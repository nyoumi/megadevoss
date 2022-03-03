<template>
  <ion-app>
    <ion-split-pane content-id="main-content">
   <!--    <ion-menu content-id="main-content" type="overlay" mode="ios">
        <ion-content>
          <ion-list id="inbox-list">
                        
            <ion-list-header>
              <div class="avatar">
                <ion-avatar>
                <img src="assets/images/logo.png">
              </ion-avatar>
              </div>

              

            </ion-list-header>
              <div class="megadevoss">MegaDeVoss</div>

            <ion-menu-toggle auto-hide="false">
              <ion-item  lines="none"  detail="false" class="hydrated"  @click="selectedIndex = -1" 
              router-direction="root" router-link="/folder/Dashboard"  >
                  <ion-icon slot="start" :md="gridSharp" :ios="gridSharp"></ion-icon>
                <ion-label>Dashboard</ion-label>
              </ion-item>
            </ion-menu-toggle>
  
            <ion-menu-toggle auto-hide="false" v-for="(p, i) in categories" :key="i">
              <ion-item @click="selectedIndex = i" router-direction="root" :router-link="'/folder/'+p.cat_id" lines="none" detail="false" class="hydrated" :class="{ selected: selectedIndex === i }">
                <ion-icon slot="start" :ios="fastFoodOutline" :md="fastFoodOutline"></ion-icon>
                <ion-label>{{ p.category_name }}</ion-label>
              </ion-item>
              
            </ion-menu-toggle>
   
            
          </ion-list>
            <ion-menu-toggle auto-hide="false">
              <ion-item  lines="none"  detail="false" class="hydrated"  @click="makeRefresh" >
                  <ion-icon slot="start" :md="refresh.icon"></ion-icon>
                <ion-label>Refresh</ion-label>
              </ion-item>
            </ion-menu-toggle>
                
          
        </ion-content>
      </ion-menu> -->
      <ion-router-outlet id="main-content"></ion-router-outlet>
    </ion-split-pane>
  </ion-app>
</template>

<script lang="ts">
import { IonApp,loadingController , IonRouterOutlet, IonSplitPane, IonicVue } from '@ionic/vue';
import { defineComponent, ref } from 'vue';
import { useRoute } from 'vue-router';
import { gridOutline,gridSharp,refreshCircleOutline,fastFoodOutline } from 'ionicons/icons';
import { Http } from '@capacitor-community/http';
import { Storage } from '@capacitor/storage';


export default defineComponent({
  name: 'App',
  components: {
    IonApp,  
    IonRouterOutlet, 
    IonSplitPane,
  },
   setup() {
    
    const selectedIndex = ref(0);
    let categories=ref();

    categories.value = [
     
      {
        category_name: 'African Dishes',
        url: '/folder/Outbox',
        cat_id: 2,
        mdIcon: fastFoodOutline
      }
    ];

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
    const refresh= {title: 'Refresh',url: '/folder/Refresh',icon: refreshCircleOutline};
    
    const path = window.location.pathname.split('folder/')[1];
    if (path !== undefined) {
      selectedIndex.value = categories.value.findIndex((page:any) => page.category_name.toLowerCase() === path.toLowerCase());
      
    }
    
    const route = useRoute();
   
    
    return { 
      selectedIndex,
      categories, 
      fastFoodOutline,
      gridOutline,gridSharp,refreshCircleOutline,
      refresh,
      isSelected: (url: string) => url === route.path ? 'selected' : ''
    }
  },
  props: {
    timeout: { type: Number, default: 100000 },
  },
  methods: {
     async makeRefresh()  {   
       this.presentLoading()
      await this.refreshMeals()
      await this.refreshCategories()

      loadingController.dismiss()



      },
       async saveData(datas:any,key:string){
         console.log("+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++")
         console.log(datas)
         await Storage.set({
          key: key,
          value: JSON.stringify(datas)
        });

      },
  /*     async downloadImg(imgName:any) {
  const options = {
    url: 'https://app.megadevoss.com/images/uploads'+imgName,
    filePath: imgName,
    //fileDirectory: Directory.Downloads,
    // Optional
    method: 'GET',
  };

  // Writes to local filesystem
  const response: HttpDownloadFileResult = await Http.downloadFile(options);
  console.log(response.blob?.size)

  // Then read the file
 /*  if (response.path) {
    const read = await Filesystem.readFile({
      path: imgName,
      //directory: Directory.Downloads,
    });
  } 
}, */
    async presentLoading() {
       const loading = await loadingController
        .create({
          cssClass: 'my-custom-class',
          message: 'Loading...',
          mode:"ios",
          backdropDismiss:true
          
        });
        
        
      await loading.present();
      
      setTimeout(function() {
        loading.dismiss()
      }, this.timeout);
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
    JsonData.forEach((element:string) => {
      let food=JSON.parse(element);
    if (food.food_id!=null) {
        console.log(food.food_img)
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
    JsonData.forEach((element:string) => {
      let cat=JSON.parse(element);
    if (cat.cat_id!=null) {
        console.log(cat.category_name)
        //this.downloadImg(food.food_img)
    }
    
  });

  }
  console.log(JSON.parse(response.data))  
  console.log("refreshing")
    }
    
    
    },
  
});
</script>

<style scoped>
ion-card{
  border-radius: 15px;
}
.avatar{
      width: 100%;
    text-align: center;
    display: flex;
    flex-direction: row;
    flex-wrap: nowrap;
    align-content: center;
    justify-content: center;
}
.megadevoss{
  text-align: center;
  color: white;
  margin-bottom: 30px;
}
ion-avatar{
  --border-radius:0;
}
ion-menu ion-content {
  --background: var(--ion-item-background, var(--ion-background-color, #141414));
}

ion-menu.md ion-content {

  background: var(--ion-item-background, var(--ion-background-color, #141414));
  background: #141414;

}

ion-menu.md ion-list {
  padding: 20px 0;
  --background: var(--ion-item-background, var(--ion-background-color, #141414));
  background: #141414;

}

ion-menu.md ion-note {
  margin-bottom: 30px;
}

ion-menu.md ion-list-header,
ion-menu.md ion-note {
  padding-left: 10px;
  color: #ffffff;
}

ion-menu.md ion-list#inbox-list {
  border-bottom: 1px solid var(--ion-color-step-150, #d7d8da);
    background: #141414;

}

ion-menu.md ion-list#inbox-list ion-list-header {
  font-size: 22px;
  font-weight: 300;
  color: #ffffff;
  min-height: 20px;
}

ion-menu.md ion-list#labels-list ion-list-header {
  font-size: 16px;

  margin-bottom: 18px;

  color: #ffffff;
  background: #141414;
  --background: var(--ion-item-background, var(--ion-background-color, #141414));

  min-height: 26px;
}

ion-menu.md ion-item {
  --padding-start: 10px;
  --padding-end: 10px;
  border-radius: 4px;
  background: #141414;
  --background: var(--ion-item-background, var(--ion-background-color, #141414));

}

ion-menu.md ion-item.selected {
  --background: #909090;
  background: #909090;
}

ion-menu.md ion-item.selected ion-icon {
    color: #ffffff;

}


ion-menu.md ion-item ion-icon {
    color: #ffffff;

}

ion-menu.md ion-item ion-label {
  font-weight: 500;
    color: #ffffff;

}

ion-menu.ios ion-content {
  --padding-bottom: 20px;
  --background: var(--ion-item-background, var(--ion-background-color, #141414));
  background: #141414;
}

ion-menu.ios ion-list {
  padding: 20px 0 0 0;
  --background: var(--ion-item-background, var(--ion-background-color, #141414));
  background: #141414;
}

ion-menu.ios ion-note {
  line-height: 24px;
  margin-bottom: 20px;
}

ion-menu.ios ion-item {
  --padding-start: 16px;
  --padding-end: 16px;
  --min-height: 50px;
  --background: var(--ion-item-background, var(--ion-background-color, #141414));
  background: #141414;
}

ion-menu.ios ion-item.selected ion-icon {
    color: #ffffff;
    

}

ion-menu.ios ion-item ion-icon {
  font-size: 24px;
  color: #ffffff;

}

ion-menu.ios ion-list#labels-list ion-list-header {
  margin-bottom: 8px;
  --background: var(--ion-item-background, var(--ion-background-color, #141414));
  background: #141414;
  color: #ffffff;
}

ion-menu.ios ion-list-header,
ion-menu.ios ion-note {
  padding-left: 16px;
  padding-right: 16px;
}

ion-menu.ios ion-note {
  margin-bottom: 8px;
}

ion-note {
  display: inline-block;
  font-size: 16px;

  color: var(--ion-color-medium-shade);
}

ion-item.selected {
  --color: var(--ion-color-primary);
}
</style>

