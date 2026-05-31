<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Juegos de Química</title>
<script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.9.3/dist/confetti.browser.min.js"></script>
<style>
    @keyframes animacionFondo { 0% { background-position: 0% 50%; } 50% { background-position: 100% 50%; } 100% { background-position: 0% 50%; } }
    @keyframes flotar { 0% { transform: translateY(0px); } 50% { transform: translateY(-15px); } 100% { transform: translateY(0px); } }
    
    .destello { position: absolute; width: 6px; height: 6px; background: #00d2ff; border-radius: 50%; pointer-events: none; box-shadow: 0 0 10px #00d2ff, 0 0 20px #ffffff; animation: desvanecer 0.5s ease-out forwards; z-index: 9999; }
    @keyframes desvanecer { 0% { transform: scale(1); opacity: 1; } 100% { transform: scale(2.5); opacity: 0; } }
    
    body { 
        margin: 0; 
        font-family: Arial, sans-serif; 
        background: linear-gradient(135deg, #141e30, #243b55, #0f2027, #203a43, #2c5364); 
        background-size: 300% 300%; 
        animation: animacionFondo 10s ease infinite; 
        color: white; 
        text-align: center; 
        overflow-x: hidden; 
        min-height: 100vh; 
        padding: 10px;
        box-sizing: border-box;
        touch-action: manipulation;
        /* --- SEGURIDAD: Prevenir selección de texto --- */
        user-select: none;
        -webkit-user-select: none;
        -moz-user-select: none;
        -ms-user-select: none;
    }
    
    .titulo-flotante { font-size: 45px; animation: flotar 3s ease-in-out infinite; margin: 0 10px 20px 10px; }
    h1 { font-size: 32px; padding: 0 10px; }
    
    .container { margin-top: 20px; display: none; width: 100%; }
    
    .caja { 
        background: white; 
        color: black; 
        width: 90%; 
        max-width: 500px; 
        margin: auto; 
        padding: 20px; 
        border-radius: 20px; 
        box-shadow: 0 0 20px rgba(0,0,0,0.5); 
        transition: all 0.5s ease; 
        box-sizing: border-box;
    }
    
    button { 
        width: 100%; 
        padding: 15px; 
        margin-top: 12px; 
        border: none; 
        border-radius: 12px; 
        font-size: 16px; 
        cursor: pointer; 
        transition: 0.3s; 
        box-sizing: border-box;
        user-select: none;
        -webkit-user-select: none;
    }
    button:hover { transform: scale(1.02); }
    
    #resultado { margin-top: 15px; font-size: 18px; font-weight: bold; }
    #contador { margin-top: 15px; font-size: 16px; line-height: 1.5; }
    
    #menuPrincipal { 
        display: flex; 
        flex-direction: column; 
        align-items: center; 
        justify-content: center; 
        min-height: 90vh; 
    }
    .menuBtn { width: 85%; max-width: 300px; padding: 18px; font-size: 20px; margin: 10px; border: none; border-radius: 15px; cursor: pointer; }

    #dinoGame { display: none; margin-top: 25px; width: 100%; }
    .canvas-container { width: 100%; max-width: 700px; margin: 15px auto; overflow-x: auto; }
    canvas { background: white; border-radius: 10px; border: 2px solid black; display: block; max-width: 100%; height: auto; }

    #credito {
        position: fixed;
        bottom: 12px;
        right: 12px;
        padding: 6px 10px;
        background: rgba(0,0,0,0.25);
        backdrop-filter: blur(4px);
        border-radius: 8px;
        font-size: 12px;
        color: rgba(255,255,255,0.85);
        font-family: Arial, sans-serif;
        z-index: 9999;
        pointer-events: none;
        box-shadow: 0 0 10px rgba(0,0,0,0.2);
    }

    @media (min-width: 768px) {
        .titulo-flotante { font-size: 55px; }
        h1 { font-size: 42px; }
        .caja { padding: 30px; }
        button { font-size: 18px; }
        #resultado { font-size: 22px; }
        #contador { font-size: 20px; }
    }
</style>
</head>

<body>

<div id="credito">
    Developed by Juan Carlos Studio © 2026
</div>

<div id="menuPrincipal">
    <h1 class="titulo-flotante">🧪 Juegos de Química</h1>
    <p style="font-size:20px; margin-bottom: 20px;">Selecciona un modo de juego</p>
    <button class="menuBtn" onclick="iniciarQuiz()">📝 Quiz</button>
    <button class="menuBtn" onclick="iniciarMemorama()">🧠 Memorama</button>
</div>

<div class="container">
    <h1>🧪 Juego de Química Orgánica</h1>
    <div class="caja">
        <h2 id="pregunta" style="font-size: 22px; margin-bottom: 15px;"></h2>
        <div id="opciones"></div>
        <div id="resultado"></div>
        <div id="contador">
            Pregunta: <span id="numero">1</span>/20<br>
            Puntos: <span id="puntos">0</span><br>
            ❤️ Vidas: <span id="vidas">5</span><br>
            ⏰ Tiempo restante: <span id="tiempo">60</span> segundos
        </div>
    </div>
    <div id="dinoGame">
        <h2>🦖 Minijuego desbloqueado</h2>
        <p>Presiona ESPACIO (PC) o TOCA LA PANTALLA para saltar</p>
        <div class="canvas-container">
            <canvas id="gameCanvas" width="700" height="200"></canvas>
        </div>
    </div>
</div>

<script>
// --- SEGURIDAD: Desactivar clic derecho ---
document.addEventListener('contextmenu', function(e) {
    e.preventDefault();
});

// --- GESTIÓN DE SONIDOS ---
const sonidos = {
    click: new Audio('https://assets.mixkit.co/active_storage/sfx/2571/2571-preview.mp3'),
    correcto: new Audio('https://assets.mixkit.co/active_storage/sfx/1435/1435-preview.mp3'),
    incorrecto: new Audio('https://assets.mixkit.co/active_storage/sfx/2569/2569-preview.mp3'),
    gameover: new Audio('https://assets.mixkit.co/active_storage/sfx/2864/2864-preview.mp3'),
    salto: new Audio('https://assets.mixkit.co/active_storage/sfx/270/270-preview.mp3')
};

function reproducirSonido(tipo) {
    if(sonidos[tipo]) {
        sonidos[tipo].currentTime = 0;
        sonidos[tipo].play().catch(() => console.log("Sonido bloqueado"));
    }
}

document.addEventListener('click', () => {
    Object.values(sonidos).forEach(s => {
        s.volume = 0.6;
        s.play().then(() => { s.pause(); s.currentTime = 0; }).catch(() => {});
    });
}, { once: true });

document.addEventListener('click', (e) => {
    if (e.target.tagName === 'BUTTON') reproducirSonido('click');
});

// --- EFECTOS VISUALES ---
const crearDestello = (x, y) => {
    let destello = document.createElement('div');
    destello.className = 'destello';
    destello.style.left = (x - 3) + 'px';
    destello.style.top = (y - 3) + 'px';
    document.body.appendChild(destello);
    setTimeout(() => { destello.remove(); }, 500);
};

document.addEventListener('mousemove', function(e) {
    if (e.buttons === 1) crearDestello(e.pageX, e.pageY);
});
document.addEventListener('touchmove', function(e) {
    if (e.touches.length === 1) crearDestello(e.touches[0].pageX, e.touches[0].pageY);
});

// --- BANCO COMPLETO DE 50 PREGUNTAS ---
const preguntas = [
    {pregunta:"¿Cuál es el principal elemento de la química orgánica?", respuesta:"Carbono", opciones:["Carbono","Hierro","Oxígeno","Sodio"]},
    {pregunta:"¿Fórmula del metano?", respuesta:"CH4", opciones:["CH4","CO2","H2O","C2H6"]},
    {pregunta:"¿Qué grupo funcional tienen los alcoholes?", respuesta:"-OH", opciones:["-OH","-COOH","-CHO","-NH2"]},
    {pregunta:"¿Qué hidrocarburo tiene enlace doble?", respuesta:"Alqueno", opciones:["Alcano","Alqueno","Alquino","Alcohol"]},
    {pregunta:"¿Cuál es un alcano?", respuesta:"Etano", opciones:["Etano","Eteno","Etino","Benceno"]},
    {pregunta:"¿Qué compuesto contiene triple enlace?", respuesta:"Alquino", opciones:["Alcano","Alcohol","Alquino","Cetona"]},
    {pregunta:"¿Cuál es la fórmula del etanol?", respuesta:"C2H5OH", opciones:["C2H5OH","CH4","CO2","H2SO4"]},
    {pregunta:"¿Qué grupo funcional tienen los ácidos carboxílicos?", respuesta:"-COOH", opciones:["-COOH","-OH","-NH2","-CHO"]},
    {pregunta:"¿Qué elemento acompaña casi siempre al carbono?", respuesta:"Hidrógeno", opciones:["Hidrógeno","Plata","Hierro","Calcio"]},
    {pregunta:"¿Qué compuesto pertenece a los alcoholes?", respuesta:"Metanol", opciones:["Metanol","Metano","Etino","Benceno"]},
    {pregunta:"¿Cuál es un alqueno?", respuesta:"Eteno", opciones:["Eteno","Etano","Metano","Propano"]},
    {pregunta:"¿Qué significa química orgánica?", respuesta:"Estudio del carbono", opciones:["Estudio del carbono","Estudio de metales","Estudio nuclear","Estudio de sales"]},
    {pregunta:"¿Qué compuesto es aromático?", respuesta:"Benceno", opciones:["Benceno","Metano","Etano","Etino"]},
    {pregunta:"¿Qué grupo funcional tienen las cetonas?", respuesta:"C=O", opciones:["C=O","-OH","-COOH","-NH2"]},
    {pregunta:"¿Cuál es el alcano más simple?", respuesta:"Metano", opciones:["Metano","Etano","Propano","Butano"]},
    {pregunta:"¿Qué tipo de enlace tiene un alcano?", respuesta:"Simple", opciones:["Simple","Doble","Triple","Iónico"]},
    {pregunta:"¿Cuál es la fórmula del eteno?", respuesta:"C2H4", opciones:["C2H4","CH4","C2H6","C3H8"]},
    {pregunta:"¿Qué grupo funcional tienen las aminas?", respuesta:"-NH2", opciones:["-NH2","-OH","-COOH","-CHO"]},
    {pregunta:"¿Qué compuesto es un alcohol?", respuesta:"Etanol", opciones:["Etanol","Etano","Eteno","Etino"]},
    {pregunta:"¿Qué hidrocarburo tiene enlace triple?", respuesta:"Alquino", opciones:["Alquino","Alqueno","Alcano","Alcohol"]},
    {pregunta:"¿Cuál es la fórmula del propano?", respuesta:"C3H8", opciones:["C3H8","C2H6","CH4","C4H10"]},
    {pregunta:"¿Cuál es la fórmula del butano?", respuesta:"C4H10", opciones:["C4H10","C3H8","C2H4","CH4"]},
    {pregunta:"¿Qué hidrocarburo pertenece a los alcanos?", respuesta:"Propano", opciones:["Propano","Propeno","Propino","Fenol"]},
    {pregunta:"¿Qué hidrocarburo pertenece a los alquenos?", respuesta:"Propeno", opciones:["Propeno","Propano","Propino","Metano"]},
    {pregunta:"¿Qué hidrocarburo pertenece a los alquinos?", respuesta:"Propino", opciones:["Propino","Propeno","Propano","Etanol"]},
    {pregunta:"¿Cuál es el grupo funcional de los aldehídos?", respuesta:"-CHO", opciones:["-CHO","-OH","-NH2","-COOH"]},
    {pregunta:"¿Cuál es el grupo funcional de los ésteres?", respuesta:"-COO-", opciones:["-COO-","-OH","-CHO","-NH2"]},
    {pregunta:"¿Qué compuesto contiene el grupo -NH2?", respuesta:"Amina", opciones:["Amina","Alcohol","Cetona","Ácido"]},
    {pregunta:"¿Qué compuesto contiene el grupo -OH?", respuesta:"Alcohol", opciones:["Alcohol","Amina","Alquino","Cetona"]},
    {pregunta:"¿Qué compuesto contiene el grupo -COOH?", respuesta:"Ácido carboxílico", opciones:["Ácido carboxílico","Alcohol","Aldehído","Amina"]},
    {pregunta:"¿Qué compuesto contiene el grupo -CHO?", respuesta:"Aldehído", opciones:["Aldehído","Alcohol","Amina","Alcano"]},
    {pregunta:"¿Cuál es la fórmula molecular del etano?", respuesta:"C2H6", opciones:["C2H6","C2H4","CH4","C3H8"]},
    {pregunta:"¿Cuál es la fórmula molecular del acetileno?", respuesta:"C2H2", opciones:["C2H2","C2H4","C2H6","CH4"]},
    {pregunta:"¿Cuál es el hidrocarburo más simple?", respuesta:"Metano", opciones:["Metano","Etano","Propano","Butano"]},
    {pregunta:"¿Qué enlace caracteriza a los alquenos?", respuesta:"Doble enlace", opciones:["Doble enlace","Triple enlace","Enlace iónico","Simple y triple"]},
    {pregunta:"¿Qué enlace caracteriza a los alquinos?", respuesta:"Triple enlace", opciones:["Triple enlace","Doble enlace","Simple enlace","Metálico"]},
    {pregunta:"¿Qué enlace caracteriza a los alcanos?", respuesta:"Enlace simple", opciones:["Enlace simple","Doble enlace","Triple enlace","Iónico"]},
    {pregunta:"¿Cuál es el principal componente del gas natural?", respuesta:"Metano", opciones:["Metano","Etano","Propano","Butano"]},
    {pregunta:"¿Qué elemento está presente en todos los compuestos orgánicos?", respuesta:"Carbono", opciones:["Carbono","Oxígeno","Hierro","Sodio"]},
    {pregunta:"¿Cuál es la valencia del carbono?", respuesta:"4", opciones:["4","2","6","8"]},
    {pregunta:"¿Qué tipo de cadena tiene forma de anillo?", respuesta:"Cíclica", opciones:["Cíclica","Lineal","Ramificada","Abierta"]},
    {pregunta:"¿Cómo se llama una cadena de carbono abierta?", respuesta:"Acíclica", opciones:["Acíclica","Cíclica","Aromática","Metálica"]},
    {pregunta:"¿Qué compuesto se usa comúnmente como combustible?", respuesta:"Propano", opciones:["Propano","Etanol","Metanol","Acetona"]},
    {pregunta:"¿Cuál es el alcohol más simple?", respuesta:"Metanol", opciones:["Metanol","Etanol","Propanol","Butanol"]},
    {pregunta:"¿Qué compuesto es una cetona?", respuesta:"Acetona", opciones:["Acetona","Metanol","Metano","Ácido acético"]},
    {pregunta:"¿Cuál es el nombre del ácido presente en el vinagre?", respuesta:"Ácido acético", opciones:["Ácido acético","Ácido sulfúrico","Ácido nítrico","Ácido clorhídrico"]},
    {pregunta:"¿Qué compuesto tiene aroma característico y anillo bencénico?", respuesta:"Benceno", opciones:["Benceno","Etano","Metanol","Propano"]},
    {pregunta:"¿Qué rama de la química estudia los compuestos del carbono?", respuesta:"Química orgánica", opciones:["Química orgánica","Química inorgánica","Bioquímica","Fisicoquímica"]},
    {pregunta:"¿Qué hidrocarburo tiene dos átomos de carbono y doble enlace?", respuesta:"Eteno", opciones:["Eteno","Etano","Etino","Metano"]},
    {pregunta:"¿Qué hidrocarburo tiene dos átomos de carbono y triple enlace?", respuesta:"Etino", opciones:["Etino","Etano","Eteno","Propano"]}
];

function mezclar(array){
    for(let i = array.length - 1; i > 0; i--){
        let j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
    }
    return array;
}

let preguntasMezcladas = mezclar([...preguntas]).slice(0, 20);
let actual = 0; let puntos = 0; let vidas = 5; let tiempo = 60; let temporizador; let temporizadorMemorama;

function iniciarQuiz(){
    document.getElementById("menuPrincipal").style.display = "none";
    document.querySelector(".container").style.display = "block";
    cargarPregunta();
}

function iniciarMemorama(){

    document.getElementById("menuPrincipal").style.display = "none";
    document.querySelector(".container").style.display = "block";

    document.querySelector(".caja").style.maxWidth = "900px";

    document.querySelector(".caja").innerHTML = `
        <h2>🧠 Memorama de Química Orgánica</h2>

        <div id="contador" style="margin-bottom:15px;">
            ⭐ Puntos: <span id="memPuntos">0</span>/10<br>
            ⏰ Tiempo: <span id="memTiempo">300</span> segundos
        </div>

        <div id="tableroMemorama"
        style="
        display:grid;
        grid-template-columns:repeat(auto-fit,minmax(100px,1fr));
        gap:10px;
        margin-top:20px;
        ">
        </div>
    `;

    const bancoPares = [
        ["Metano", "CH4"],
        ["Etano", "C2H6"],
        ["Propano", "C3H8"],
        ["Butano", "C4H10"],
        ["Pentano", "C5H12"],
        ["Hexano", "C6H14"],
        ["Heptano", "C7H16"],
        ["Octano", "C8H18"],
        ["Nonano", "C9H20"],
        ["Decano", "C10H22"],
        ["Eteno", "C2H4"],
        ["Etino", "C2H2"],
        ["Benceno", "C6H6"],
        ["Metanol", "CH3OH"],
        ["Etanol", "C2H5OH"],
        ["Propanol", "C3H7OH"],
        ["Alcohol", "Grupo -OH"],
        ["Amina", "Grupo -NH2"],
        ["Aldehído", "Grupo -CHO"],
        ["Ácido carboxílico", "Grupo -COOH"],
        ["Éster", "Grupo -COO-"],
        ["Cetona", "Grupo C=O"],
        ["Alcano", "Enlace simple"],
        ["Alqueno", "Doble enlace"],
        ["Alquino", "Triple enlace"],
        ["Carbono", "Valencia 4"],
        ["Química orgánica", "Estudio del carbono"],
        ["Cadena abierta", "Acíclica"],
        ["Cadena cerrada", "Cíclica"],
        ["Acetona", "Removedor de esmalte"],
        ["Fenol", "Alcohol aromático"],
        ["Alcohol de madera", "Metanol (tóxico)"],
        ["Combustible doméstico", "Gas LP"],
        ["Acetileno", "Alquino de soldadura"],
        ["Primer alqueno", "Etileno"],
        ["Aromático", "Anillo bencénico"],
        ["Éter", "R-O-R'"],
        ["Amida", "Grupo -CONH2"],
        ["Haluro", "R-X"],
        ["Polímero", "Macromolécula"]
    ];

    const pares = mezclar([...bancoPares]).slice(0,10);

    let cartas = [];

    pares.forEach((par,id)=>{
        cartas.push({ texto:par[0], pareja:id });
        cartas.push({ texto:par[1], pareja:id });
    });

    cartas = mezclar(cartas);

    const tablero = document.getElementById("tableroMemorama");

    let primeraCarta = null;
    let segundaCarta = null;
    let bloqueado = false;

    let puntosMemorama = 0;
    let tiempoMemorama = 300;

    temporizadorMemorama = setInterval(()=>{

        tiempoMemorama--;

        let tiempoElemento = document.getElementById("memTiempo");
        if(tiempoElemento){
            tiempoElemento.innerHTML = tiempoMemorama;
        }

        if(tiempoMemorama <= 0){

            clearInterval(temporizadorMemorama);

            reproducirSonido("gameover");

            document.querySelector(".caja").innerHTML = `
                <h2>⏰ Tiempo agotado</h2>
                <h1>${puntosMemorama}/10</h1>
                <button onclick="location.reload()">🔄 Reiniciar</button>
            `;
        }

    },1000);

    cartas.forEach((carta,index)=>{

        const boton = document.createElement("button");

        boton.innerHTML = "🧬⚗️";

        boton.style.height = "90px";
        boton.style.fontSize = "16px";
        boton.dataset.pareja = carta.pareja;
        boton.dataset.texto = carta.texto;
        boton.dataset.abierta = "false";

        boton.onclick = function(){

            if(bloqueado) return;
            if(boton.dataset.abierta === "true") return;

            boton.innerHTML = carta.texto;
            boton.dataset.abierta = "true";

            if(!primeraCarta){
                primeraCarta = boton;
                return;
            }

            segundaCarta = boton;
            bloqueado = true;

            if(primeraCarta.dataset.pareja === segundaCarta.dataset.pareja){

                reproducirSonido("correcto");
                puntosMemorama++;
                
                let puntosElemento = document.getElementById("memPuntos");
                if(puntosElemento) puntosElemento.innerHTML = puntosMemorama;

                primeraCarta.style.background = "#b8ffb8";
                segundaCarta.style.background = "#b8ffb8";

                primeraCarta = null;
                segundaCarta = null;
                bloqueado = false;

                if(puntosMemorama === 10){
                    clearInterval(temporizadorMemorama);
                    lanzarConfeti();

                    let caja = document.querySelector(".caja");
                    caja.style.background = "transparent"; 
                    caja.style.boxShadow = "none"; 
                    caja.style.color = "white";
                    
                    caja.innerHTML = `<h1 style="font-size: 35px; margin-top: 20px;">🏆 ¡Memorama Perfecto! 🎉</h1>`;

                    setTimeout(() => {
                        caja.style.background = "white"; 
                        caja.style.boxShadow = "0 0 20px rgba(0,0,0,0.5)"; 
                        caja.style.color = "black";

                        caja.innerHTML = `
                            <h2>🏆 Memorama completado</h2>
                            <h1>10/10</h1>
                            <h2>🔥 Desbloqueaste el minijuego</h2>
                            <p>Excelente trabajo en química orgánica 🧪</p>
                        `;

                        iniciarDinoGame('memorama');
                    }, 1500);
                }

            }else{

                reproducirSonido("incorrecto");

                setTimeout(()=>{
                    primeraCarta.innerHTML = "🧬⚗️";
                    segundaCarta.innerHTML = "🧬⚗️";
                    primeraCarta.dataset.abierta = "false";
                    segundaCarta.dataset.abierta = "false";
                    primeraCarta = null;
                    segundaCarta = null;
                    bloqueado = false;
                },900);
            }

        };

        tablero.appendChild(boton);

    });
}

function cargarPregunta(){
    clearInterval(temporizador);
    tiempo = 60; document.getElementById("tiempo").innerHTML = tiempo;
    temporizador = setInterval(() => {
        tiempo--; document.getElementById("tiempo").innerHTML = tiempo;
        if(tiempo <= 0){
            clearInterval(temporizador);
            reproducirSonido('incorrecto');
            document.getElementById("resultado").innerHTML = "⏰ Tiempo agotado";
            vidas--; document.getElementById("vidas").innerHTML = vidas;
            actual++;
            setTimeout(() => { if(vidas <= 0) terminarJuego(); else if(actual < preguntasMezcladas.length) cargarPregunta(); else terminarJuego(); }, 1000);
        }
    },1000);
    
    document.getElementById("resultado").innerHTML = "";
    let preguntaActual = preguntasMezcladas[actual];
    document.getElementById("pregunta").innerHTML = preguntaActual.pregunta;
    document.getElementById("numero").innerHTML = actual + 1;
    
    let opciones = mezclar([...preguntaActual.opciones]);
    let contenedor = document.getElementById("opciones");
    contenedor.innerHTML = "";
    
    opciones.forEach(opcion => {
        let boton = document.createElement("button");
        boton.innerHTML = opcion;
        boton.onclick = function(){ verificarRespuesta(opcion); };
        contenedor.appendChild(boton);
    });
}

function verificarRespuesta(opcion){
    clearInterval(temporizador);
    let correcta = preguntasMezcladas[actual].respuesta;
    if(opcion === correcta){
        reproducirSonido('correcto');
        document.getElementById("resultado").innerHTML = "✅ Correcto";
        puntos++;
    }else{
        reproducirSonido('incorrecto');
        document.getElementById("resultado").innerHTML = "❌ Incorrecto<br>Respuesta correcta: " + correcta;
        vidas--; document.getElementById("vidas").innerHTML = vidas;
    }
    document.getElementById("puntos").innerHTML = puntos;
    actual++;
    setTimeout(() => { if(vidas <= 0) terminarJuego(); else if(actual < preguntasMezcladas.length) cargarPregunta(); else terminarJuego(); }, 1200);
}

function lanzarConfeti() {
    let duration = 1.5 * 1000; let animationEnd = Date.now() + duration;
    let defaults = { startVelocity: 30, spread: 360, ticks: 60, zIndex: 1000 };
    function randomInRange(min, max) { return Math.random() * (max - min) + min; }
    let interval = setInterval(function() {
        let timeLeft = animationEnd - Date.now();
        if (timeLeft <= 0) return clearInterval(interval);
        let particleCount = 50 * (timeLeft / duration);
        confetti(Object.assign({}, defaults, { particleCount, origin: { x: randomInRange(0.1, 0.3), y: Math.random() - 0.2 } }));
        confetti(Object.assign({}, defaults, { particleCount, origin: { x: randomInRange(0.7, 0.9), y: Math.random() - 0.2 } }));
    }, 250);
}

function terminarJuego(){
    clearInterval(temporizador);
    if(vidas <= 0) reproducirSonido('gameover');
    let caja = document.querySelector(".caja");
    if(vidas <= 0){
        caja.innerHTML = `<h2>💀 Te quedaste sin vidas</h2><h1>${puntos}/20</h1><p>Necesitas mínimo 15 puntos</p><button onclick="location.reload()">🔄 Reiniciar juego</button>`;
    }else if(puntos >= 15){
        if(puntos === 20){
            lanzarConfeti();
            caja.style.background = "transparent"; caja.style.boxShadow = "none"; caja.style.color = "white";
            caja.innerHTML = `<h1 style="font-size: 35px; margin-top: 20px;">🏆 ¡Puntuación Perfecta! 🎉</h1>`;
            setTimeout(() => {
                caja.style.background = "white"; caja.style.boxShadow = "0 0 20px rgba(0,0,0,0.5)"; caja.style.color = "black";
                caja.innerHTML = `<h2>🏆 ¡Puntuación Perfecta! 🎉</h2><h1>${puntos}/20</h1><h2>🔥 Desbloqueaste el minijuego</h2><p>Eres bueno en química orgánica 🧪</p>`;
                
                iniciarDinoGame('quiz');
            }, 1500);
        } else {
            caja.innerHTML = `<h2>🎉 Juego terminado</h2><h1>${puntos}/20</h1><h2>🔥 Desbloqueaste el minijuego</h2><p>Eres bueno en química orgánica 🧪</p>`;
            
            iniciarDinoGame('quiz');
        }
    }else{
        caja.innerHTML = `<h2>🎉 Juego terminado</h2><h1>${puntos}/20</h1><p>Necesitas mínimo 15 puntos</p><button onclick="location.reload()">🔄 Reiniciar juego</button>`;
    }
}

// --- MINIJUEGO ---
function iniciarDinoGame(modo = 'quiz'){
    document.getElementById("dinoGame").style.display = "block";
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    let dino = {x:50, y:150, width:40, height:40, vy:0, jumping:false};
    let gravedad = 0.8; let velocidad = 6; let puntosDino = 0; let obstaculos = [];
    
    let obstaculoInterval = setInterval(crearObstaculo, 1500);
    
    function crearObstaculo(){ obstaculos.push({x:700, y:160, width:20 + Math.random()*20, height:20 + Math.random()*40}); }
    function dibujarDino(){ ctx.fillStyle = "green"; ctx.fillRect(dino.x, dino.y, dino.width, dino.height); }
    
    function actualizar(){
        ctx.clearRect(0,0,canvas.width,canvas.height);
        ctx.fillStyle = "black"; ctx.font = "20px Arial"; ctx.fillText("Puntos: " + puntosDino,20,30);
        dino.vy += gravedad; dino.y += dino.vy;
        if(dino.y >= 150){ dino.y = 150; dino.vy = 0; dino.jumping = false; }
        dibujarDino();
        
        for(let i = 0; i < obstaculos.length; i++){
            let obs = obstaculos[i]; obs.x -= velocidad;
            ctx.fillStyle = "red"; ctx.fillRect(obs.x, obs.y, obs.width, obs.height);
            
            if(dino.x < obs.x + obs.width && dino.x + dino.width > obs.x && dino.y < obs.y + obs.height && dino.y + dino.height > obs.y){
                clearInterval(obstaculoInterval);
                reproducirSonido('gameover');
                
                let puntosPrevios = "";
                if(modo === 'quiz') {
                    puntosPrevios = `<h2>🧪 Puntos del quiz: ${puntos}/20</h2>`;
                } else if(modo === 'memorama') {
                    puntosPrevios = `<h2>🧠 Puntos del memorama: 10/10</h2>`;
                } else {
                    puntosPrevios = `<h2>🛠️ Modo desarrollador activo</h2>`;
                }

                document.body.innerHTML = `<div style="font-family:Arial; text-align:center; margin-top:100px; color:white; padding:20px;"><h1>💀 GAME OVER</h1>${puntosPrevios}<h2>🦖 Puntos del dinosaurio: ${puntosDino}</h2><button onclick="location.reload()" style="padding:20px; font-size:20px; border:none; border-radius:12px; cursor:pointer; margin-top:20px; width:100%; max-width:300px;">🔄 Reiniciar juego</button></div>`;
                return;
            }
            if(obs.x + obs.width < 0){ 
                obstaculos.splice(i,1); 
                i--;
                puntosDino++; 
            }
        }
        velocidad += 0.002; requestAnimationFrame(actualizar);
    }
    
    const ejecutarSalto = () => {
        if(!dino.jumping){
            reproducirSonido('salto');
            dino.vy = -12;
            dino.jumping = true;
        }
    };

    document.onkeydown = function(e){ 
        if(e.code === "Space") {
            ejecutarSalto(); 
        }
    };
    
    document.getElementById("dinoGame").addEventListener("touchstart", function(e){ 
        e.preventDefault(); 
        ejecutarSalto(); 
    }, {passive: false});
    
    actualizar();
}

</script>
</body>
</html>
