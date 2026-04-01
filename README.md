<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Cognitive AI</title>

<style>
body {
  margin:0;
  font-family: Arial;
  background: linear-gradient(135deg,#1f1c2c,#928dab);
  color:white;
}
.container { max-width:500px; margin:auto; padding:10px;}
.chat-box { height:300px; overflow-y:auto; background:rgba(255,255,255,0.1); padding:10px; border-radius:10px;}
.msg { padding:8px; margin:5px; border-radius:8px; max-width:80%;}
.user { background:#4facfe; margin-left:auto;}
.ai { background:#43e97b; color:black;}
input, button { width:100%; padding:10px; margin-top:5px; border:none; border-radius:10px;}
button { background:white; }
</style>
</head>

<body>
<div class="container">

<h2>🧠 Cognitive AI (Stable Version)</h2>

<input id="goal" placeholder="Your goal">
<button onclick="saveGoal()">Save Goal</button>

<div id="chat" class="chat-box"></div>

<input id="input" placeholder="Type message">
<button onclick="send()">Send</button>

</div>

<script>

// MEMORY (LOCAL SAFE)
function getMemory(){
 return JSON.parse(localStorage.getItem("memory")||"[]");
}
function saveMemory(m){
 localStorage.setItem("memory", JSON.stringify(m));
}

// UI
function addMessage(text,type){
 let div=document.createElement("div");
 div.className="msg "+type;
 div.innerText=text;
 chat.appendChild(div);
 chat.scrollTop=chat.scrollHeight;
}

// GOAL
function saveGoal(){
 localStorage.setItem("goal", goal.value);
 alert("Goal saved");
}

// 🧠 OFFLINE FALLBACK AI (ALWAYS WORKS)
function fallbackAI(input){
 let goal=localStorage.getItem("goal")||"";

 if(input.toLowerCase().includes("hello"))
   return "Hello! How can I help you?";

 if(input.toLowerCase().includes("goal"))
   return "Focus daily on: "+goal;

 if(goal)
   return "To achieve '"+goal+"', start with: "+input;

 return "I understand: "+input;
}

// 🌐 REAL AI (SAFE)
async function realAI(input){

 try{
  let res = await fetch("https://openrouter.ai/api/v1/chat/completions",{
   method:"POST",
   headers:{
    "Authorization":"Bearer YOUR_API_KEY",
    "Content-Type":"application/json"
   },
   body:JSON.stringify({
    model:"mistralai/mistral-7b-instruct",
    messages:[{role:"user",content:input}]
   })
  });

  let data = await res.json();

  if(data && data.choices){
    return data.choices[0].message.content;
  } else {
    throw "API failed";
  }

 }catch(err){
   console.log("Fallback used:",err);
   return fallbackAI(input);
 }
}

// SEND
async function send(){

 let text=input.value;
 if(!text) return;

 addMessage(text,"user");
 addMessage("Thinking...","ai");

 let reply = await realAI(text);

 chat.lastChild.remove();

 addMessage("🧠 "+reply,"ai");

 let memory=getMemory();
 memory.push({user:text, ai:reply});
 saveMemory(memory);

 input.value="";
}

</script>
</body>
</html>
