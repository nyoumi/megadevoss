<template>
  <ion-page>
    <div class="sidebar" v-bind:class="{ sidebarVisible: isActive }" >
    <div class="logo-details">
      <img src="assets/images/logo.png" style="width: 85px;" alt="">
      <span class="menuinvisible"  v-bind:class="{ menuvisible: isActive }" >MegaDeVoss</span>
        <ion-button color="transparent"  class="bouton" slot="start" v-bind:class="{ menuinvisible: !isActive }" @click="openMenu()">
          <ion-icon :ios="menu" :md="menu" color="primary"></ion-icon></ion-button>

    </div>
      <ul class="nav-links">
        <li>

          <a href="/folder/Dashboard"  >
            <ion-icon slot="start" :md="gridSharp" :ios="gridSharp" color="primary" class="bx bx-box"></ion-icon>
            <span class="links_name menuinvisible" v-bind:class="{ menuvisible: isActive }">Dashboard</span>
          </a>
        </li>
        <li v-for="(p, i) in categories" :key="i" :router-link="'/folder/'+p.cat_id">
                  <a :href="'/folder/'+p.cat_id" >

                 <ion-icon slot="start" :ios="fastFood" :md="fastFood" color="primary" class="bx bx-box"></ion-icon>
                <ion-label class="links_name menuinvisible"  v-bind:class="{ menuvisible: isActive }" >{{ p.category_name }}</ion-label>
                </a>
        </li>
        <li @click="makeRefresh">
          <a >
            <ion-icon slot="start" :md="refreshCircleOutline" :ios="refreshCircleOutline" color="primary" class="bx bx-box"></ion-icon>
            <span class="links_name menuinvisible" v-bind:class="{ menuvisible: isActive }">Refresh</span>
          </a>
        </li> 
        <li @click="openAdmin">
          <a>
            <ion-icon slot="start" :md="personCircleOutline" :ios="personCircleOutline" color="primary" class="bx bx-box"></ion-icon>
            <span class="links_name menuinvisible" v-bind:class="{ menuvisible: isActive }">Admin</span>
          </a>
        </li>       
      </ul>
  </div>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-buttons slot="start">
          <ion-button color="primary" slot="start" @click="openMenu()"><ion-icon :ios="menu" :md="menu"></ion-icon></ion-button>
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
        <ion-backdrop v-bind:class="{ menuinvisible: !isActive }"
    :tappable="isActive"
    :visible="!isActive"
     @ionBackdropTap="openMenu()">
  </ion-backdrop>
      
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
import { IonButtons, IonContent, IonHeader,IonIcon,loadingController,IonBackdrop, IonPage,IonButton, IonToolbar,IonSearchbar,IonBadge,IonCard,IonAvatar,IonImg, onIonViewWillEnter } from '@ionic/vue';
import { useRouter,useRoute } from 'vue-router';
import { Storage } from '@capacitor/storage';
import { fastFood,gridSharp,menu,refreshCircleOutline,personCircleOutline } from 'ionicons/icons';
import { Http } from '@capacitor-community/http';
import { Browser } from '@capacitor/browser';


export default defineComponent({
  name: 'FolderPage',
  components: {
    IonButtons,
    IonContent,
    IonHeader,
    IonButton,
    IonPage,
    IonBackdrop,
    IonIcon,
    IonToolbar,
    IonSearchbar,IonBadge,IonCard,IonAvatar,
    IonImg
  },
    setup() {
      const meals=ref();
      const allMeals=ref()
      const route = useRoute();
      const { id } = route.params;
      const isActive= ref();
      isActive.value=true;
      console.log(id)

      let categories=ref();

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
     (async() => {  
      console.log('App vue page will enter');
       const { value } = await Storage.get({ key: 'categories' })
       
         if(value!=null) {
           let datas=JSON.parse(value);
           let temp:any=[]
           datas.forEach((element:string) => {
             if (JSON.parse(element).cat_id!=null) {
                temp.push(JSON.parse(element))
             }
             
           });
           categories.value=temp;
           
         }
         console.log(categories.value) 
    })();
    return {
      router: useRouter(),
      meals,
      allMeals,
      categories,
      fastFood,
      gridSharp,refreshCircleOutline,
      personCircleOutline,
      isActive,
      menu
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
  },
  openMenu(){
    this.isActive= (!this.isActive)
    console.log(this.isActive)
  },
       async makeRefresh()  {   
       this.presentLoading()
      await this.refreshMeals()
      await this.refreshCategories()
      loadingController.dismiss()
      },
       async presentLoading() {
       const loading = await loadingController
        .create({
          cssClass: 'my-custom-class',
          message: 'Loading...',
          mode:"ios",
          backdropDismiss:true
        });
        
        
      await loading.present();
      
    },
    async refreshMeals(){
             const options = {
    url: 'https://app.megadevoss.com/api.php/foods',
    //headers: { 'X-Fake-Header': 'Max was here' },
    //params: { size: 'XL' },
  };


  const response = await Http.request({ ...options, method: 'GET' }) 
  if(response.status==200){
    let JsonData=JSON.parse(response.data)
    this.saveData(JsonData,'meals');
    this.meals=[]
    JsonData.forEach((element:string) => {
      let food=JSON.parse(element);
    if (food.food_id!=null) {
        console.log(food.food_img)
        this.meals.push(food)
        //this.downloadImg(food.food_img)
    }
    
  });

  }
  console.log(JSON.parse(response.data))  
  console.log("refreshing")
    },
        async refreshCategories(){
             const options = {
    url: 'https://app.megadevoss.com/api.php/categories',
    //headers: { 'X-Fake-Header': 'Max was here' },
    //params: { size: 'XL' },
  };


  const response = await Http.request({ ...options, method: 'GET' }) 
  if(response.status==200){
    let JsonData=JSON.parse(response.data)
    this.saveData(JsonData,'categories');
    this.categories=[]
    JsonData.forEach((element:string) => {
      let cat=JSON.parse(element);
    if (cat.cat_id!=null) {
        console.log(cat.category_name)
        this.categories.push(cat)        //this.downloadImg(food.food_img)
    }
    
  });

  }
  console.log(JSON.parse(response.data))  
  console.log("refreshing")
    },
    
  async saveData(datas:any,key:string){
    console.log("+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++")
    console.log(datas)
    await Storage.set({
    key: key,
    value: JSON.stringify(datas)
  });

      },
      async openAdmin() {
        await Browser.open({ url: 'https://app.megadevoss.com/' });
      },
    }
  
  
});
</script>

<style scoped>
.bouton{
  --box-shadow: unset;
}
.sidebarVisible{
  width: 250px !important;
}
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
    padding-bottom: 100px;
    margin-left: 65px;
    text-align: center;
    margin-top: 2%;
}
ion-card {
    margin-left: 2%;
    /* padding-left: 10px; */
    margin-right: 2%;
    display: inline-flex;
    width: 90%;
    max-width: 300px;
    margin-inline: 65px 10px 10px 20px;
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
    padding: 11px 9px 22px 0px;
}
ion-header{
  margin-left: 60px !important; 
  width: auto !important;
}

/**beginning */
.detail-price-tag {
  position: absolute;
  font-size: 20px;
  font-weight: 600;
  color: #ffffff;
  text-align: left;
  bottom: 51px;
  left: 25px;
  background-color: #ff0000;
  border-radius: 5px;
  padding-top: 3px;
  padding-bottom: 3px;
  padding-left: 11px;
  padding-right: 11px;
}

.responsive .hero-image {
  background-image: linear-gradient(rgba(0, 0, 0, 0.5), rgba(0, 0, 0, 0.5));
  width: 90%;
  height: auto;
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
  position: relative;
}

.responsive {
  width: 100%;
  height: auto;
  border-radius: 11px;
}

.overview-boxes .price-tag {
  position: absolute;
  font-size: 14px;
  font-weight: 600;
  color: #ffffff;
  text-align: left;
  bottom: 8px;
  right: 10px;
  background-color: #ff0000;
  border-radius: 5px;
  padding-top: 3px;
  padding-bottom: 3px;
  padding-left: 2px;
  padding-right: 2px;
}

/* Googlefont Poppins CDN Link */
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@200;300;400;500;600;700&display=swap');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Poppins', sans-serif;
}

.sidebar {
  position: fixed;
  height: 100%;
  width: 60px;
  z-index: 10000;
  background: #141414;
  transition: all 0.5s ease;
  border-right: 3px solid rgb(58, 58, 58);
  overflow-y:auto;
}

.sidebar.active {
  width: 60px;
}

.sidebar .logo-details {
  height: 70px;
  display: flex;
  color: #fff;
}

.sidebar .logo-details i {
  font-size: 20px;
  font-weight: 400;
  color: #fff;
  min-width: 60px;
}

.sidebar .logo-details .logo_name {
  color: #fff;
  font-size: 24px;
  font-weight: 500;
}

.sidebar .nav-links {
  margin-top: 10px;
  padding-left: 20px;
}

.sidebar .nav-links li {
  position: relative;
  list-style: none;
  height: 50px;
}

.sidebar .nav-links li a {
  height: 100%;
  width: 100%;
  display: flex;
  align-items: center;
  text-decoration: none;
  transition: all 0.4s ease;
}

.sidebar .nav-links li a.active {
  background: #909090;
}

.sidebar .nav-links li a:hover {
  background: #909090;
}

.sidebar .nav-links li i {
  min-width: 60px;
  text-align: center;
  font-size: 18px;
  color: #fff;
}

.sidebar .nav-links li a .links_name {
  color: rgb(255, 255, 255);
  font-size: 15px;
  font-weight: 400;
  white-space: nowrap;
  margin-left: 20px;
}

.sidebar .nav-links .log_out {
  position: absolute;
  bottom: 0;
  width: 100%;
}

.home-section {
  position: relative;
  background: #000000;
  min-height: 100vh;
  width: calc(100% - 240px);
  left: 240px;
  transition: all 0.5s ease;
}

.sidebar.active~.home-section {
  width: calc(100% - 60px);
  left: 60px;
}

.home-section nav {
  display: flex;
  justify-content: space-between;
  height: 80px;
  background: #141414;
  display: flex;
  align-items: center;
  position: fixed;
  width: calc(100% - 240px);
  left: 240px;
  z-index: 100;
  padding: 0 20px;
  box-shadow: 0 1px 1px rgb(0, 0, 0);
  transition: all 0.5s ease;
}

.sidebar.active~.home-section nav {
  left: 60px;
  width: calc(100% - 60px);
}

.home-section nav .sidebar-button {
  display: flex;
  align-items: center;
  font-size: 24px;
  font-weight: 500;
  color: #ffffff;
}

nav .sidebar-button i {
  font-size: 35px;
  margin-right: 10px;
}

.home-section nav .search-box {
  position: relative;
  height: 50px;
  max-width: 500px;
  width: 90%;
  margin: 0 20px;
}

nav .search-box input {
  height: 100%;
  width: 100%;
  outline: none;
  background: #c6c6c7;
  border: 2px solid #EFEEF1;
  border-radius: 6px;
  font-size: 18px;
  padding: 0 15px;
}

nav .search-box .bx-search {
  position: absolute;
  height: 40px;
  width: 40px;
  background: #000000;
  right: 5px;
  top: 50%;
  transform: translateY(-50%);
  border-radius: 4px;
  line-height: 40px;
  text-align: center;
  color: #fff;
  font-size: 22px;
  transition: all 0.4 ease;
}

.home-section nav .profile-details {
  display: flex;
  align-items: center;
  background: #c6c6c7;
  border: 2px solid #EFEEF1;
  border-radius: 6px;
  height: 50px;
  min-width: 190px;
  padding: 0 15px 0 2px;
}

nav .profile-details img {
  height: 40px;
  width: 40px;
  border-radius: 6px;
  object-fit: cover;
}

nav .profile-details .admin_name {
  font-size: 15px;
  font-weight: 500;
  color: rgb(0, 0, 0);
  margin: 0 10px;
  white-space: nowrap;
}

nav .profile-details i {
  font-size: 25px;
  color: rgb(0, 0, 0);
}

.home-section .home-content {
  position: relative;
  padding-top: 104px;
}

.home-content .overview-boxes {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  padding: 0 20px;
  margin-bottom: 26px;
}

.overview-boxes .box {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: calc(100% / 4 - 15px);
  background: #040707;
  padding: 5px 1px;
  border-radius: 11px;
  box-shadow: 0 5px 10px rgb(0, 0, 0);
}

.overview-boxes .box-topic {
  position: absolute;
  font-size: 17px;
  font-weight: 500;
  color: #ffffff;
  text-align: left;
  top: 4px;
  left: 10px;
  padding: 5px;
  text-shadow: 2px 2px rgb(0, 0, 0);
}

.home-content .box .number {
  display: inline-block;
  font-size: 35px;
  margin-top: -6px;
  font-weight: 500;
}

.home-content .box .indicator {
  display: flex;
  align-items: center;
}

.home-content .box .indicator i {
  height: 20px;
  width: 20px;
  background: #8FDACB;
  line-height: 20px;
  text-align: center;
  border-radius: 50%;
  color: #fff;
  font-size: 20px;
  margin-right: 5px;
}

.box .indicator i.down {
  background: #e87d88;
}

.home-content .box .indicator .text {
  font-size: 12px;
}

.home-content .box .cart {
  display: inline-block;
  font-size: 32px;
  height: 50px;
  width: 50px;
  background: #cce5ff;
  line-height: 50px;
  text-align: center;
  color: #66b0ff;
  border-radius: 12px;
  margin: -15px 0 0 6px;
}

.home-content .box .cart.two {
  color: #2BD47D;
  background: #C0F2D8;
}

.home-content .box .cart.three {
  color: #ffc233;
  background: #ffe8b3;
}

.home-content .box .cart.four {
  color: #e05260;
  background: #f7d4d7;
}

.home-content .total-order {
  font-size: 20px;
  font-weight: 500;
}

.home-content .sales-boxes {
  display: flex;
  justify-content: space-between;
  /* padding: 0 20px; */
}

/* left box */
.home-content .sales-boxes .recent-sales {
  width: 100%;
  background: #fff;
  padding: 20px 30px;
  margin: 0 0px;
  border-radius: 12px;
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.1);
}

.home-content .sales-boxes .sales-details {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-bottom: 60px;
}

.sales-boxes .box .title {
  font-size: 24px;
  font-weight: 500;
  /* margin-bottom: 10px; */
}

.sales-boxes .sales-details li.topic {
  font-size: 20px;
  font-weight: 500;
}

.sales-boxes .sales-details li {
  list-style: none;
  margin: 8px 0;
}

.sales-boxes .sales-details li a {
  font-size: 18px;
  color: #333;
  font-size: 400;
  text-decoration: none;
}

.sales-boxes .box .button {
  width: 100%;
  display: flex;
  justify-content: flex-end;
}

.sales-boxes .box .button a {
  color: #fff;
  background: #0A2558;
  padding: 4px 12px;
  font-size: 15px;
  font-weight: 400;
  border-radius: 4px;
  text-decoration: none;
  transition: all 0.3s ease;
}

.sales-boxes .box .button a:hover {
  background: #0d3073;
}

/* Right box */
.home-content .sales-boxes .top-sales {
  width: 35%;
  background: #fff;
  padding: 20px 30px;
  margin: 0 20px 0 0;
  border-radius: 12px;
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.1);
}

.sales-boxes .top-sales li {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin: 10px 0;
}

.sales-boxes .top-sales li a img {
  height: 40px;
  width: 40px;
  object-fit: cover;
  border-radius: 12px;
  margin-right: 10px;
  background: #333;
}

.sales-boxes .top-sales li a {
  display: flex;
  align-items: center;
  text-decoration: none;
}

.sales-boxes .top-sales li .product,
.price {
  font-size: 17px;
  font-weight: 400;
  color: #333;
}

.sidebar .nav-links li a {
    height: 100%;
    width: 100%;
    display: flex;
    align-items: center;
    text-decoration: none;
    transition: all 0.4s ease;
    flex-direction: row;
    flex-wrap: nowrap;
    align-content: center;
}

.menuvisible{
  display: inline !important;
}
.menuinvisible{
  display: none;
}
ion-page{
  margin-left: 60px;
}

/* Responsive Media Query */
@media (max-width: 1240px) {
  .sidebar {
    width: 60px;
  }

  .sidebar.active {
    width: 220px;
  }

  .home-section {
    width: calc(100% - 60px);
    left: 60px;
  }

  .sidebar.active~.home-section {
    /* width: calc(100% - 220px); */
    overflow: hidden;
    left: 220px;
  }

  .home-section nav {
    width: calc(100% - 60px);
    left: 60px;
  }

  .sidebar.active~.home-section nav {
    width: calc(100% - 220px);
    left: 220px;
  }
}

@media (max-width: 1150px) {
  .home-content .sales-boxes {
    flex-direction: column;
    font-size: 15px;
  }

  .home-content .sales-boxes .box {
    width: 100%;
    overflow-x: scroll;
    margin-bottom: 30px;
  }

  .home-content .sales-boxes .top-sales {
    margin: 0;
  }
}

@media (max-width: 1000px) {
  .overview-boxes .box {
    width: calc(100% / 2 - 15px);
    margin-bottom: 15px;
    font-size: 11px;
  }
}

@media (max-width: 700px) {

  nav .sidebar-button .dashboard,
  nav .profile-details .admin_name,
  nav .profile-details i {
    display: none;
    font-size: 9px;
  }

  .home-section nav .profile-details {
    height: 50px;
    min-width: 40px;
  }

  .home-content .sales-boxes .sales-details {
    width: 560px;
  }
}

@media (max-width: 550px) {
  .overview-boxes .box {
    width: 100%;
    margin-bottom: 15px;
  }

  .sidebar.active~.home-section nav .profile-details {
    display: none;
  }
  .bx-box{
    margin-right: 20px;
  }
}
</style>
