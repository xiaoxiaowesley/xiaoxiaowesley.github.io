
## My Awesome Page

<div align="center">
 <label class="switch">
 <input type="checkbox" id="language-switch">
 <span class="slider round"></span>
 </label>
</div>

<div id="english-content">

 ### English Version

 Hello, and welcome to my awesome page! Here you'll find all sorts of interesting information about my life, my hobbies, and my thoughts on the world.

 ### About Me

 I'm a software engineer based in San Francisco, California. I love to code, read, and travel. In my free time, you can usually find me exploring new hiking trails or trying out new restaurants.

 ### My Hobbies

 I have a lot of hobbies, but my favorites are:

 - Reading
 - Hiking
 - Cooking
 - Playing video games
</div>

<div id="french-content" style="display: none;">

### French Version

 Bonjour et bienvenue sur ma page géniale ! Ici, vous trouverez toutes sortes d'informations intéressantes sur ma vie, mes passe-temps et mes réflexions sur le monde.

 ### À propos de moi

 Je suis un ingénieur logiciel basé à San Francisco, en Californie. J'aime coder, lire et voyager. Pendant mon temps libre, vous pouvez généralement me trouver en train d'explorer de nouveaux sentiers de randonnée ou d'essayer de nouveaux restaurants.

 ### Mes passe-temps

 J'ai beaucoup de passe-temps, mais mes préférés sont :

 - Lire
 - Randonnée
 - Cuisiner
 - Jouer à des jeux vidéo
 
</div>

<script>
 const languageSwitch = document.querySelector('#language-switch');
 const englishContent = document.querySelector('#english-content');
 const frenchContent = document.querySelector('#french-content');

 languageSwitch.addEventListener('change', () => {
 if (languageSwitch.checked) {
 englishContent.style.display = 'none';
 frenchContent.style.display = 'block';
 } else {
 englishContent.style.display = 'block';
 frenchContent.style.display = 'none';
 }
 });
</script>

<style>
 .switch {
 position: relative;
 display: inline-block;
 width:60px;
 height:34px;
 }

 .switch input {
 opacity:0;
 width:0;
 height:0;
 }

 .slider {
 position: absolute;
 cursor: pointer;
 top:0;
 left:0;
 right:0;
 bottom:0;
 background-color: #ccc;
 -webkit-transition: .4s;
 transition: .4s;
 }

 .slider:before {
 position: absolute;
 content: "";
 height:26px;
 width:26px;
 left:4px;
 bottom:4px;
 background-color: white;
 -webkit-transition: .4s;
 transition: .4s;
 }

 input:checked + .slider {
 background-color: #2196F3;
 }

 input:focus + .slider {
 box-shadow:001px #2196F3;
 }

 input:checked + .slider:before {
 -webkit-transform: translateX(26px);
 -ms-transform: translateX(26px);
 transform: translateX(26px);
 }

 .slider.round {
 border-radius:34px;
 }

 .slider.round:before {
 border-radius:50%;
 }
</style>