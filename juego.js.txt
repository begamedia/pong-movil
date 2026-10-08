const canvas = document.getElementById("lienzo");
const ctx = canvas.getContext("2d");

// variables globales equivalentes a tus atributos de java
let estadoInicio = 0; // 0=menu, 2=juego, 3=fin
let nivelCampaña = 1;
let contadorRebotesRaqueta = 0;
let cronometroLetreroNivel = 0;
let ganadorReal = 1;
let enRevancha = false;

// estructuras de los objetos del juego
let pelota = { x: 368, y: 110, tam: 28, velX: 0, velY: 0, cayendo: false };
let jugador1 = { x: 30, y: 240, ancho: 18, alto: 120, velY: 0 }; // maquina
let jugador2 = { x: 752, y: 240, ancho: 18, alto: 120, velY: 0 }; // humano
let marcador = { j1: 0, j2: 0 };

// inicializa el saque de cortesía de la ronda activa
function reiniciarPelota(jugadorQueAnoto) {
    pelota.x = 386;
    pelota.y = 286;
    let base = enRevancha ? 5.8 : 4.5;
    pelota.velX = (jugadorQueAnoto === 1) ? base : -base;
    pelota.velY = (Math.random() > 0.5) ? base : -base;
}

// evalua la transicion continua de niveles al cumplir la meta
function evaluarTransicionContinua() {
    if (contadorRebotesRaqueta >= 15) {
        contadorRebotesRaqueta = 0;
        cronometroLetreroNivel = 100;

        if (nivelCampaña === 1) {
            nivelCampaña = 2;
            pelota.velX *= 1.2;
        } else if (nivelCampaña === 2) {
            nivelCampaña = 3;
            pelota.velX *= 1.4;
        } else if (nivelCampaña === 3) {
            if (marcador.j1 === marcador.j2) {
                nivelCampaña = 4;
                enRevancha = true;
                pelota.velX *= 1.6;
            } else {
                estadoInicio = 3;
                ganadorReal = (marcador.j1 < marcador.j2) ? 2 : 1;
            }
        }
    }
}

// bucle de fisica y logica del motor interactivo
function actualizar() {
    if (estadoInicio === 0) {
        // animacion suave de espera en el menu principal
        pelota.y += 1;
        if (pelota.y > 140) pelota.y = 110;
    }

    if (estadoInicio === 2) {
        // movimiento de paleta del humano (jugador 2)
        jugador2.y += jugador2.velY;
        if (jugador2.y < 60) jugador2.y = 60;
        if (jugador2.y > 420) jugador2.y = 420;

        // ia humana balanceada para el jugador 1 (maquina izquierda)
        if (pelota.x < 400) {
            let centroRaqueta = jugador1.y + 60;
            if (pelota.y > centroRaqueta + 15) jugador1.y += 5;
            else if (pelota.y < centroRaqueta - 15) jugador1.y -= 5;
        }
        if (jugador1.y < 60) jugador1.y = 60;
        if (jugador1.y > 420) jugador1.y = 420;

        // movimiento de la pelota
        pelota.x += pelota.velX;
        pelota.y += pelota.velY;

        // rebotes contra el techo y suelo
        if (pelota.y <= 60) {
            pelota.y = 60;
            pelota.velY = Math.abs(pelota.velY) * (enRevancha ? (0.8 + Math.random()*0.5) : 1);
        }
        if (pelota.y >= 512) {
            pelota.y = 512;
            pelota.velY = -Math.abs(pelota.velY) * (enRevancha ? (0.8 + Math.random()*0.5) : 1);
        }

        // colision con raqueta 1 (maquina)
        if (pelota.x <= jugador1.x + jugador1.ancho && pelota.y + 28 >= jugador1.y && pelota.y <= jugador1.y + jugador1.alto) {
            if (pelota.velX < 0) {
                pelota.velX = Math.abs(pelota.velX);
                contadorRebotesRaqueta++;
                evaluarTransicionContinua();
            }
        }

        // colision con raqueta 2 (humano)
        if (pelota.x + 28 >= jugador2.x && pelota.y + 28 >= jugador2.y && pelota.y <= jugador2.y + jugador2.alto) {
            if (pelota.velX > 0) {
                pelota.velX = -Math.abs(pelota.velX);
                contadorRebotesRaqueta++;
                evaluarTransicionContinua();
            }
        }

        // control de cronometro del texto flotante
        if (cronometroLetreroNivel > 0) cronometroLetreroNivel--;

        // control de goles y perdida de corazones
        if (pelota.x <= 0) {
            marcador.j1++; // le quita vida a la maquina
            if (marcador.j1 >= 11) { estadoInicio = 3; ganadorReal = 2; }
            else reiniciarPelota(2);
        }
        if (pelota.x >= 772) {
            marcador.j2++; // le quita vida al humano
            if (marcador.j2 >= 11) { estadoInicio = 3; ganadorReal = 1; }
            else reiniciarPelota(1);
        }
    }
}

// renderizado de los degradados y graficos vhs
function dibujar() {
    ctx.clearRect(0, 0, 800, 600);

    // calculo del degradado invertido con la franja ochenta por ciento oscura
    let colorClaro = "#822d02"; // naranja oscuro quemado nivel 1
    if (nivelCampaña === 2) colorClaro = "#0f5014"; // verde oliva
    if (nivelCampaña === 3) colorClaro = "#3c0f5a"; // violeta quemado
    if (nivelCampaña === 4) colorClaro = "#6e0808"; // rojo sangre

    let degradado = ctx.createLinearGradient(0, 0, 0, 600);
    degradado.addColorStop(0, "#050a1e");
    degradado.addColorStop(0.65, "#050a1e");
    degradado.addColorStop(1, colorClaro);
    ctx.fillStyle = degradado;
    ctx.fillRect(0, 0, 800, 600);

    // red divisoria sutil
    ctx.fillStyle = "rgba(255,255,255,0.15)";
    for (let i = 60; i < 600; i += 30) ctx.fillRect(398, i, 4, 15);

    // dibujo de raquetas y pelota fluorescente
    ctx.fillStyle = "#ffffff";
    ctx.fillRect(jugador1.x, jugador1.y, jugador1.ancho, jugador1.alto);
    ctx.fillRect(jugador2.x, jugador2.y, jugador2.ancho, jugador2.alto);

    ctx.fillStyle = "#32ff32"; // verde fluorescente
    ctx.beginPath();
    ctx.arc(pelota.x + 14, pelota.y + 14, 14, 0, Math.PI * 2);
    ctx.fill();

    ctx.fillStyle = "#ffffff";
    ctx.font = "bold 20px monospace";

    // interfaz del menu o del panel de juego activo
    if (estadoInicio === 0) {
        ctx.fillStyle = "#00f0dc"; // aguamarina
        ctx.font = "bold 64px monospace";
        ctx.drawString = ctx.fillText("PONG", 320, 140);

        ctx.fillStyle = "#ffe600"; // amarillo activo
        ctx.strokeRect(300, 260, 200, 50);
        ctx.font = "bold 18px monospace";
        ctx.fillText("TOCA PARA JUGAR", 325, 292);
    }

    if (estadoInicio === 2) {
        ctx.fillText("MAQUINA " + "♥".repeat(11 - marcador.j1), 50, 40);
        ctx.fillText("JUGADOR " + "♥".repeat(11 - marcador.j2), 430, 40);

        if (cronometroLetreroNivel > 0 && cronometroLetreroNivel % 20 > 5) {
            ctx.fillStyle = "#ffe600";
            ctx.font = "bold 46px monospace";
            let txt = "NIVEL " + nivelCampaña;
            if (nivelCampaña === 4) txt = "REVANCHA";
            ctx.fillText(txt, 280, 260);
        }
    }

    if (estadoInicio === 3) {
        ctx.font = "bold 42px monospace";
        ctx.fillStyle = (ganadorReal === 2) ? "#32ff32" : "#ff0000";
        let txtFin = (ganadorReal === 2) ? "HUMANO GANADOR" : "MAQUINA GANADORA";
        ctx.fillText(txtFin, 200, 240);
        ctx.font = "16px monospace";
        ctx.fillStyle = "#ffe600";
        ctx.fillText("TOCA PARA REINICIAR", 290, 320);
    }
}

// bucle principal del juego animado por el navegador
**window.requestAnimationFrame** (function ciclo() {
    actualizar();
    dibujar();
    window.requestAnimationFrame(ciclo);
});

// --- CAPTURA DE EVENTOS TACTILES MÓVILES PARA UX ---
const btnArriba = document.getElementById("btn-arriba");
const btnAbajo = document.getElementById("btn-abajo");

btnArriba.addEventListener("touchstart", (e) => { e.preventDefault(); jugador2.velY = -6; });
btnArriba.addEventListener("touchend", () => jugador2.velY = 0);
btnAbajo.addEventListener("touchstart", (e) => { e.preventDefault(); jugador2.velY = 6; });
btnAbajo.addEventListener("touchend", () => jugador2.velY = 0);

// detector de toques en la pantalla para avanzar los menus
canvas.addEventListener("click", () => {
    if (estadoInicio === 0) {
        estadoInicio = 2;
        reiniciarPelota(1);
    } else if (estadoInicio === 3) {
        estadoInicio = 0;
        marcador.j1 = 0; marcador.j2 = 0;
        nivelCampaña = 1; enRevancha = false;
        pelota.x = 368; pelota.y = 110;
    }
});
