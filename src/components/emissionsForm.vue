<template lang="html">
    <article>
        <section id="button-container"> 
            <button class="button" v-if="!transport.status" v-on:click="handleClickTransport">Transport</button>
            <button class="button" v-if="!diet.status" v-on:click="handleClickDiet">Diet</button>
            <button class="button" v-if="!energy.status" v-on:click="handleClickEnergy()">Energy</button> 
        </section>

         <section>
            <div class="transport-form base-form" v-if="transport.status">

            <p id="small-heading">Distance travelled in miles</p>
            <form class="transport" v-on:submit="addTransportData" method="post">
                <label for="transport-type">Transport Type:</label>
                <select id="transport-dropDown" required v-model="transport.type">
                    <option value="" disabled selected >Select transport type</option>
                    <option v-for="(carbon, label, index) in transportTypes" :key="index" :value="label">{{label}}</option>
                </select>

                <label for="transport-quantity">Quantity (miles):</label>
                <input type="number" required id="transport-quantity-input" class="inputs" v-model.number="transport.quantity"/> 

                <input type="submit" value="Submit Transport" class="transport-input-button" id="save"/>     
            </form>
            </div>
         </section>

         <section>
            <div class="diet-form base-form" v-if="diet.status">
        
            <form class="diet" v-on:submit="addDietData" method="post">
                <label for="diet-select" id="small-heading"> Select a Diet Type</label>

                <select required name="diet-select" id="diet-select" class="inputs-diet" v-model="selectedDiet">
                    <option value="" disabled selected >Select diet type</option>
                    <option v-for="(carbon,label,index) in diets" :key="index" :value="label"> {{label}} </option>
                </select>
          
                <input type="submit" value="Submit Diet" class="diet-input-button" id="save" />
                
            </form>
            </div>
         </section>

         <section>
           <div class="energy-form base-form" v-if="energy.status">
                <p id="small-heading">Enter amount of energy used</p>
                <form class="energy" v-on:submit="addEnergyData" method="post">
                   <label for="energy-type">Energy</label>
                   <select  id="energy-type-dropDown"required v-model="energy.type">
                       <option value="" disabled selected >Select energy type</option>
                     <option v-for="(carbon, label, index) in energyTypes" :key="index" :value="label">{{label}}</option> 
                   </select>
                   <label for="enrgy-quantity">Quantity (kWh):</label>
                   <input type="number" required id="energy-quantity-input" class="inputs" v-model.number="energy.quantity"/> 
                   <input  type="submit" value="Submit Energy" class="energy-input-button" id="save" />
                </form>
            </div>
         </section>

    </article>
</template>

<script>
import {eventBus} from '@/main.js'
import userData from '@/services/userData.js'
import emissionGrid from '@/components/emissionGrid.vue'
import emissionFactor from '@/services/emissionsDataServices'

export default {
    name: 'emissions-form',
    

    data(){
        return{
            diets: [],
            transportTypes: [],
            energyTypes: [],
            selectedDiet: null,

            transport: {
                quantity: 0,
                type: null,
                status: false
            },

            energy: {
                quantity: 0,
                type: null,
                status: false
            },

            diet: {
               
                status: false
            }
            
        }
    },

    component: {
        'emissions-grid' : emissionGrid,
    },
   
    methods: {
        addTransportData(evt){
            evt.preventDefault()
            const transport = {
                label: this.transport.type,
                quantity: this.transport.quantity,
                unit: "mile(s)"
            }
        
            userData.postUserData(transport)
            .then(res => eventBus.$emit('transport-emissions', res))
            this.transport = {
                label: null,
                quantity: 0,
                status: false
            }
        },
        addEnergyData(evt){
            evt.preventDefault()
            const energy = { 
                label: this.energy.type,
                quantity: this.energy.quantity,
                unit: "kWh",
            }
        
            userData.postUserData(energy)
            .then(res => eventBus.$emit('energy-emissions', res))
            this.energy = {
                label: null,
                quantity: 0,
                status: false
            }
        },
        addDietData(evt){
            evt.preventDefault()
            userData.postUserData({
                label: this.selectedDiet,
                quantity: 1,
                unit: "day"
            })
            .then(res => eventBus.$emit('diet-emissions', res))
            this.diet = {
                status: false
            }
        },
        handleClickTransport: function(index) {
            this.transport.status = true
        },
        handleClickDiet: function() {
            this.diet.status = true
        },
        handleClickEnergy: function() {
            this.energy.status = true
        },
        handleSelect: function() {
        emissionFactor.getEmissionFactor(index)
            .then(res => eventBus.$emit('diet-selected', this.factor, index, res))
        },

        getEmissionFactors() {
            fetch("http://localhost:3000/api/emission/")
            .then(res => res.json())
            .then(data => {
                this.diets = data[0].diet
                this.transportTypes = data[0].transport
                this.energyTypes = data[0].energy
            })
        }


        
    },
    mounted() {
        this.getEmissionFactors();
        
    },
    
}
</script>

<style scope>

.transport-form{
    padding: 10px;
    margin: 10px 7px;
    display: flex;
    flex-flow: column wrap;
    align-items: center;
    background-color: whitesmoke ;
    border-radius: 10px;
}
.transport{
    display: flex;
    flex-flow: column wrap;
    align-items: center;
    
}
.transport-form > p {
    margin-bottom: 7px;
}
.transport-form > h2 {
    margin:0;
}

#transport-type{
    padding: 7px;
    border-radius: 7px;
}
#transport-dropDown{
    border: 1px gainsboro solid;
    padding: 7px 17px;
    border-radius: 7px;
}
#transport-quantity-input{
    border: 1px gainsboro solid;
    padding: 7px 15px;
    border-radius: 7px;
}

#diet-select{
    border: 1px gainsboro solid;
    padding: 7px 25px;
    border-radius: 7px;
}


.diet-form{
    margin: 10px 7px;
    padding: 20px;
    font-size: 18px;
    text-align: center;
    background-color: whitesmoke ;
    border-radius: 10px;
       
}
.diet {
    margin: 6px 6px;
    display: flex;
    flex-flow: column wrap;
    align-items: center;
}

#energy-type-dropDown{
    border: 1px gainsboro solid;
    padding: 7px 20px;
    border-radius: 7px;
}


#energy-quantity-input{
    border: 1px gainsboro solid;
    padding: 7px 12px;
    border-radius: 7px;
}

.energy-form{
    margin: 10px 7px;
    padding: 15px;
    text-align: center;
    background-color: whitesmoke ;
    border-radius: 10px;
}
.energy{
    display: flex;
    flex-flow: column wrap;
    align-items: center;
}

.energy-input-button{
   margin: 5px 5px;
   border: 1px solid #ccc;
   border-radius: 8px;
   color: #fff;
   padding: 10px 35px;
   background-color: #246C5A;

}
.transport-input-button{
   margin: 5px 5px;
   border: 1px solid #ccc;
   border-radius: 8px;
   color: #fff;
   padding: 10px 30px;
   background-color: #246C5A;

}

.diet-input-button{
   margin: 5px 5px;
   border: 1px solid #ccc;
   border-radius: 8px;
   color: #fff;
   padding: 10px 40px;
   background-color: #246C5A;

}
 .button{
   font-size: 15px;
   margin-top: 2px;
   border: none;
   border-radius: 7px;
   color: #fff;
   padding: 7px 30px;
   background-color: #319e2f;
 }

 .button:hover{
     background-color: #288632;
 }

 #button-container{
     margin-top: 30px;
     margin-bottom: 30px;
     display: flex;
     flex-flow: row wrap;
     justify-content: space-around;
 }
 #small-heading{
     font-weight: 400;
     font-size: 20px;
     color: #17491c;
     font-family:  Times, 'Times New Roman', serif;
 }
</style>