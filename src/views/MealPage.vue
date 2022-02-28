<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-back-button></ion-back-button>
        </ion-buttons>
                <ion-buttons slot="end">
                  <ion-avatar>
                      <img src="assets/images/me.jpg">
                  </ion-avatar>
                </ion-buttons>
       <ion-title>{{ meal.food_name }}</ion-title>
      </ion-toolbar>
    </ion-header>
    
    <ion-content :fullscreen="true">
      
      <div class="meals">
        <ion-card  class="meal">
            
          <ion-img :src="'https://template.megadevoss.com/vendors/images/uploads/'+meal.food_img"></ion-img>
          <ion-badge>{{meal.food_price}} Frs</ion-badge>
          <span class="meal-title">{{meal.food_name}} </span>
          <ion-card-content>{{meal.food_description}}</ion-card-content>
      </ion-card>
      </div>

    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import { defineComponent, ref  } from 'vue';
import { IonButtons, IonContent, IonHeader, IonPage, IonToolbar,
IonBackButton,IonBadge,IonCard,IonAvatar,IonImg,IonTitle, onIonViewWillEnter } from '@ionic/vue';
import { Storage } from '@capacitor/storage';
import { useRoute  } from 'vue-router';

export default defineComponent({
  name: 'FolderPage',
  components: {
    IonButtons,
    IonContent,
    IonHeader,
    IonPage,
    IonToolbar,
    IonBackButton,IonBadge,IonCard,IonAvatar,IonImg,
    IonTitle
      },
    setup() {
      const meal=ref({});
      const route = useRoute();
      const { id } = route.params;
      console.log(meal);

    onIonViewWillEnter(async() => {
      console.log('meal page will enter');
      const { value } = await Storage.get({ key: 'meals' })

       
         if(value!=null) {

           let datas=JSON.parse(value);
           datas.forEach((element:string) => {
             let food=JSON.parse(element);
             if (food.food_id==id) {
                console.log(food)
                meal.value=food;
             }
             
           });
           
         }
         console.log(meal.value) 
    });    
    
    return {
      meal
    }
  }
});
</script>

<style scoped>
span.meal-title {
    position: absolute;
    top: 2%;
    left: 2%;
    font-size: 17px;
    font-weight: 500;
    color: #ffffff;
    text-align: left;
    text-shadow: 2px 2px rgb(0 0 0);
}
ion-badge {
    position: absolute;
    bottom: 1%;
    right: 1%;
    --background: #ff0000;
    color: white;

}
.meals {
    align-items: center;
    text-align: center;
    margin-top: 2%;
}
ion-card {
    margin-left: auto;
    margin-right: auto;
    width: 90%;
    flex-direction: row;
    flex-wrap: nowrap;
    align-content: center;
    justify-content: center;
    align-items: center;
    padding-bottom: 20px;
}

ion-avatar{
  --border-radius: 0;
    width: 45px;
    height: 45px;
    padding: 2px; 
    display: flex;
    align-items: center;
    background: #c6c6c7;
    border: 2px solid #EFEEF1;
    border-radius: 6px;
}

ion-buttons.buttons-last-slot.sc-ion-buttons-md-h.sc-ion-buttons-md-s.md {
    margin-left: 10px;
}
#container {
  text-align: center;
  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;
  color: #8c8c8c;
  margin: 0;
}

#container a {
  text-decoration: none;
}
ion-avatar{
  --border-radius:0;
}
ion-content{
  --background: #000;
    background: #000;
    display: inline-flex;
    flex-direction: row;
    flex-wrap: wrap;
    align-content: center;
    justify-content: center;
    align-items: center;
}
ion-header ion-toolbar:first-of-type {
    padding-top: var(--ion-safe-area-top, 0);
    background: #141414;
    --background: #141414;
}
</style>
