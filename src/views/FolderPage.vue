<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-menu-button color="primary"></ion-menu-button>
        </ion-buttons>
        <ion-searchbar placeholder="Search..." showCancelButton="never" @ionChange="filterItems($event)"></ion-searchbar>
                <ion-buttons slot="end">
                  <ion-avatar>
                      <img src="assets/images/me.jpg">
                  </ion-avatar>
                </ion-buttons>


        <!-- <ion-title>{{ $route.params.id }}</ion-title> -->
      </ion-toolbar>
    </ion-header>
    
    <ion-content :fullscreen="true">
      
      <div class="meals">
        

      <ion-card v-for="(p, i) in meals" :key="i" class="meal"  @click="() => router.push('/meal/'+p.food_id)" >
        <div>
          <ion-img :src="'https://template.megadevoss.com/vendors/images/uploads/'+p.food_img"></ion-img>
          <ion-badge>{{p.food_price}} Frs</ion-badge>
          <span class="meal-title">{{p.food_name}} </span>  

        </div>
      

      </ion-card>
      </div>

    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';
import { IonButtons, IonContent, IonHeader, IonMenuButton, IonPage, IonToolbar,IonSearchbar,IonBadge,IonCard,IonAvatar,IonImg, onIonViewWillEnter } from '@ionic/vue';
import { useRouter,useRoute } from 'vue-router';
import { Storage } from '@capacitor/storage';

export default defineComponent({
  name: 'FolderPage',
  components: {
    IonButtons,
    IonContent,
    IonHeader,
    IonMenuButton,
    IonPage,
    IonToolbar,
    IonSearchbar,IonBadge,IonCard,IonAvatar,
    IonImg
  },
    setup() {
      const meals=ref();
      const allMeals=ref()
      const route = useRoute();
      const { id } = route.params;
      console.log(id)



    onIonViewWillEnter(async() => {
       const { value } = await Storage.get({ key: 'meals' })
       
         if(value!=null) {
           
           

           let datas=JSON.parse(value);
           let temp:any=[]
           datas.forEach((element:string) => {
            let food=JSON.parse(element)
             if (food.food_id!=null && (food.food_cat_id==id || id=='Dashboard')) {
                temp.push(JSON.parse(element))
             }
             
           });
           meals.value=temp;
           allMeals.value=temp;
           
         }
         console.log(meals.value) 
    });    
    
    return {
      router: useRouter(),
      meals,
      allMeals
    }
  },
  
  methods: {
    filterItems(searchTerm:any):any {
      this.meals=this.allMeals

      let searchValue:string=searchTerm.detail.value
      if (searchValue.length==0) {
        return
      }
      this.meals= this.meals.filter((item:any) => {
      console.log(searchValue.toLowerCase())
      return item.food_name.toLowerCase().indexOf(searchValue.toLowerCase()) > -1;
    });
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
    margin-left: unset;
    margin-right: unset;
    display: inline-flex;
    width: 100%;
    max-width: 300px;
    margin-inline: 10px;
    flex-direction: row;
    flex-wrap: nowrap;
    align-content: center;
    justify-content: center;
    align-items: center;
}
ion-searchbar{
  --background: #c6c6c7;
    border: 2px solid #EFEEF1;
    border-radius: 6px;
    font-size: 18px;
    padding: 0 0;
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
    padding: 15px 20px 15px;
}
</style>

function meals(meals: any) {
  throw new Error('Function not implemented.');
}
