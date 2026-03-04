<script setup>
import {ref, watch} from 'vue';

const nsfwContent = ref(false);
const maxTagCount = 100;
const anTime = 600, waitTime = 900;
const anFrames = 50;
const tag = ref(["", ""]);
const loading = ref(true), firstLoad = ref(false); 
const href = ref(["", ""]);
const count = ref(["", ""]);
const countAn = ref(["???", "???"]);

const er = ref([false, false]);
const score = ref("0"), maxScore = ref("0");

var imageCash = [{
  "sfw":{
    status: 0,
    src:""
  },
  "nsfw":{
    status: 0,
    src:""
  }
},{
  "sfw":{
    status: 0,
    src:""
  },
  "nsfw":{
    status: 0,
    src:""
  }
}];

function rndGen(){
  return Math.ceil(Math.random() * maxTagCount);
}
async function updater(isCorrect){
  if(loading.value == false){ return;}
  loading.value = false;

  if(firstLoad.value){
    await Promise.all([
      countAnimator(0),
      countAnimator(1)
    ]);
  }

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
  await Promise.all([update(p1, 0), update(p2, 1)]);  

  resetImageCash(0);
  resetImageCash(1);

  firstLoad.value = true;
  loading.value = true;
  return;
}

function resetImageCash(prop){
  imageCash[prop]["nsfw"]["status"] = 0;
  imageCash[prop]["sfw"]["status"] = 0;
}

async function countAnimator(prop){
  for(var i = 0; i < anFrames; i++){
    await new Promise(function(resolve){
      setTimeout(function(){
        countAn.value[prop] = Math.ceil((i + 1) * (count.value[prop] / anFrames));
        resolve(null);
      }, anTime / anFrames)
    });
  }

  await new Promise(function(resolve){
    setTimeout(function(){
      resolve(null);
    }, waitTime)
  });

  countAn.value[prop] = "???";
  return;
}

async function imageLoader(){
  if(loading.value == false){ return;}
  loading.value = false; 

  await Promise.all([imageLoad(0), imageLoad(1)]);

  loading.value = true;
  return;
}

async function imageLoad(prop){
  //console.log(imageCash[prop][nsfwContent.value ? "nsfw" : "sfw"]);
  if(imageCash[prop][nsfwContent.value ? "nsfw" : "sfw"].status == 0){
    var postF = await fetch(`https://api.rule34.gg/getRandomPosts?amount=1&tags=${tag.value[prop]}` + (nsfwContent.value ? "" : "&tags=sfw"));
    var post = await postF.json();
  
    if(post["posts"].length > 0){
      href.value[prop] = post["posts"][0]["preview"]["file"];
      er.value[prop] = false;
      imageCash[prop][nsfwContent.value ? "nsfw" : "sfw"].src = post["posts"][0]["preview"]["file"];
      imageCash[prop][nsfwContent.value ? "nsfw" : "sfw"].status = 2;
    }
    else{
      href.value[prop] = "";
      er.value[prop] = true;
      imageCash[prop][nsfwContent.value ? "nsfw" : "sfw"].status = 1;
    }
  }
  else if(imageCash[prop][nsfwContent.value ? "nsfw" : "sfw"].status == 1){
    href.value[prop] = "";
    er.value[prop] = true;
  }
  else{
    href.value[prop] = imageCash[prop][nsfwContent.value ? "nsfw" : "sfw"].src;
    er.value[prop] = false;
  }
}

async function update(pos, prop){
  var tagFetch = await fetch(`https://api.rule34.gg/getTags?raw=true&limit=${maxTagCount + 10}&type=is_character`);
  var tagList = await tagFetch.json();
  
  tag.value[prop] = tagList["tags"][pos]["tag_name"];
  count.value[prop] = tagList["tags"][pos]["count"];
  
  await imageLoad(prop);
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
        <button @click="updater(count[0] >= count[1])" :disabled="!loading" id="firstButton">
          <h1> {{ tag[0] }} </h1>
          <h1> {{ countAn[0] }}</h1>
          <img :src="href[0]" alt="Image couldn't load" v-if="!er[0]"/>
          <h1 v-else> Image couldn't load </h1>
        </button>
      </div>
      <div id="secondChoose">
        <button @click="updater(count[1] >= count[0])" :disabled="!loading" id="secondButton">
          <h1> {{ tag[1] }}</h1>
          <h1> {{ countAn[1] }}</h1>
          <img :src="href[1]"  alt="Image couldn't load" v-if="!er[1]"/>
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
