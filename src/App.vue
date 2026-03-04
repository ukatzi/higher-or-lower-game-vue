<script setup>
import {ref, watch} from 'vue';

const nsfwContent = ref(false);
const count = 100;
const anTime = 600, waitTime = 900;
const anFrames = 50;
const tag1 = ref(""), tag2 = ref(""), loading = ref(true), firstLoad = ref(false); 
const href1 = ref(""), href2 = ref("");
const count1 = ref(""), count2 = ref("");
const countAn1 = ref("???"), countAn2 = ref("???");

const er1 = ref(false), er2 = ref(false);
const score = ref("0"), maxScore = ref("0");

var imageCash1 = {
  "sfw":{
    status: 0,
    src:""
  },
  "nsfw":{
    status: 0,
    src:""
  }
};
var imageCash2 = {
  "sfw":{
    status: 0,
    src:""
  },
  "nsfw":{
    status: 0,
    src:""
  }
};

function rndGen(){
  return Math.ceil(Math.random() * count);
}
async function updater(isCorrect){
  if(loading.value == false){ return;}
  loading.value = false;

  if(firstLoad.value){
    await Promise.all([
      countAnimator1(),
      countAnimator2()
    ]);
  }

  console.log(123);

  if(isCorrect){
    score.value++;
  }
  else{
    score.value = 0;
  }

  maxScore.value = Math.max(maxScore.value, score.value);

  var p1 = rndGen(), p2 = rndGen();
  while(p2 == p1){
    p2 = rndGen();
  }
  await Promise.all([updater1(p1), updater2(p2)]);  

  resetImageCash();
  await Promise.all([imageLoader1(), imageLoader2()]);

  firstLoad.value = true;
  loading.value = true;
  return;
}

function resetImageCash(){
  imageCash1["nsfw"]["status"] = 0;
  imageCash1["sfw"]["status"] = 0;
  imageCash2["nsfw"]["status"] = 0;
  imageCash2["sfw"]["status"] = 0;
}

async function countAnimator1(){
  for(var i = 0; i < anFrames; i++){
    await new Promise(function(resolve){
      setTimeout(function(){
        countAn1.value = Math.ceil((i + 1) * (count1.value / anFrames));
        resolve(null);
      }, anTime / anFrames)
    });
  }

  await new Promise(function(resolve){
    setTimeout(function(){
      resolve(null);
    }, waitTime)
  });

  countAn1.value = "???";
  return;
}
async function countAnimator2(){
  for(var i = 0; i < anFrames; i++){
    await new Promise(function(resolve){
      setTimeout(function(){
        countAn2.value = Math.ceil((i + 1) * (count2.value / anFrames));
        resolve(null);
      }, anTime / anFrames)
    });
  }

  await new Promise(function(resolve){
    setTimeout(function(){
      resolve(null);
    }, waitTime)
  });

  countAn2.value = "???";
  return;
}

async function imageLoader(){
  if(loading.value == false){ return;}
  loading.value = false; 

  await Promise.all([imageLoader1(), imageLoader2()]);

  loading.value = true;
  return;
}

async function imageLoader1(){
  //console.log(imageCash1[nsfwContent.value ? "nsfw" : "sfw"]);
  if(imageCash1[nsfwContent.value ? "nsfw" : "sfw"].status == 0){
    var postF = await fetch(`https://api.rule34.gg/getRandomPosts?amount=1&tags=${tag1.value}` + (nsfwContent.value ? "" : "&tags=sfw"));
    var post = await postF.json();
  
    if(post["posts"].length > 0){
      href1.value = post["posts"][0]["preview"]["file"];
      er1.value = false;
      imageCash1[nsfwContent.value ? "nsfw" : "sfw"].src = post["posts"][0]["preview"]["file"];
      imageCash1[nsfwContent.value ? "nsfw" : "sfw"].status = 2;
    }
    else{
      href1.value = "";
      er1.value = true;
      imageCash1[nsfwContent.value ? "nsfw" : "sfw"].status = 1;
    }
  }
  else if(imageCash1[nsfwContent.value ? "nsfw" : "sfw"].status == 1){
    href1.value = "";
    er1.value = true;
  }
  else{
    href1.value = imageCash1[nsfwContent.value ? "nsfw" : "sfw"].src;
    er1.value = false;
  }
}

async function imageLoader2(){
  if(imageCash2[nsfwContent.value ? "nsfw" : "sfw"].status == 0){
    var postF = await fetch(`https://api.rule34.gg/getRandomPosts?amount=1&tags=${tag2.value}` + (nsfwContent.value ? "" : "&tags=sfw"));
    var post = await postF.json();
  
    if(post["posts"].length > 0){
      href2.value = post["posts"][0]["preview"]["file"];
      er2.value = false;
      imageCash2[nsfwContent.value ? "nsfw" : "sfw"].src = post["posts"][0]["preview"]["file"];
      imageCash2[nsfwContent.value ? "nsfw" : "sfw"].status = 2;
    }
    else{
      href2.value = "";
      er2.value = true;
      imageCash2[nsfwContent.value ? "nsfw" : "sfw"].status = 1;
    }
  }
  else if(imageCash2[nsfwContent.value ? "nsfw" : "sfw"].status == 1){
    href2.value = "";
    er2.value = true;
  }
  else{
    href2.value = imageCash2[nsfwContent.value ? "nsfw" : "sfw"].src;
    er2.value = false;
  }
}

async function updater1(pos){
  var tagFetch = await fetch(`https://api.rule34.gg/getTags?raw=true&limit=${count + 10}&type=is_character`);
  var tagList = await tagFetch.json();
  
  tag1.value = tagList["tags"][pos]["tag_name"];
  count1.value = tagList["tags"][pos]["count"];
  
  await imageLoader1();
}

async function updater2(pos){
  var tagFetch = await fetch(`https://api.rule34.gg/getTags?raw=true&limit=${count + 10}&type=is_character`);
  var tagList = await tagFetch.json();  
  
  tag2.value = tagList["tags"][pos]["tag_name"];
  count2.value = tagList["tags"][pos]["count"];
  
  await imageLoader2();
}
updater();

watch(nsfwContent, imageLoader);

</script>

<template>
    <div id="score">
      <div>
        Score: {{ score }} <br>
        Max Score: {{ maxScore }}
      </div>
    </div>
    <div v-if="firstLoad" id="buttons"> 
      <div id="firstChoose">
        <button @click="updater(count1 >= count2)" :disabled="!loading" id="firstButton">
          <h1> {{ tag1 }} </h1>
          <h1> {{ countAn1 }}</h1>
          <img :src="href1" alt="Image couldn't load" v-if="!er1"/>
          <h1 v-else> Image couldn't load </h1>
        </button>
      </div>
      <div id="secondChoose">
        <button @click="updater(count2 >= count1)" :disabled="!loading" id="secondButton">
          <h1> {{ tag2 }}</h1>
          <h1> {{ countAn2 }}</h1>
          <img :src="href2"  alt="Image couldn't load" v-if="!er2"/>
          <h1 v-else> Image couldn't load </h1>
        </button>
      </div>
    </div>
    <div v-else>
      Loading...
    </div>

  <div id ="toggles">
    Toggles <br>
    <button v-on:click="nsfwContent = !nsfwContent" :disabled="!loading"> 
      <h1>
        NSFW is <a color="red" font-size="70px">{{nsfwContent ? "ON" : "OFF" }}</a>
      </h1>
    </button>

  </div>
  <div>
    
  </div>
</template>

<style scoped>
  #firstChoose{
    text-align: center;
    
  }

  #secondChoose{
    text-align: center;
  }

  #toggles{
    position: fixed;
    top: 10px;
    left: 10px;
    height: 80px;
  }

  #firstButton{
    position: fixed;
    top: 30%;
    left: 28%;

    height: 60%;
    width: 20%;
  }
  #firstButton img{
    display: block;
    min-width: 100%;
    max-width: 100%;
    height: auto;
  }

  #secondButton{
    position: fixed;
    top: 30%;
    left: 52%;

    height: 60%;
    width: 20%;
  }
  #secondButton img{
    display: block;
    min-width: 100%;
    max-width: 100%;
    height: auto;
  }

  #score{
    position: fixed;
    top: 20%;
    left: 48%;
  }

  #buttons{
    justify-content: space-around;
  }
</style>
