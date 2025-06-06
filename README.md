# Gestor-de-Expedicioness
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Gestor de Expediciones RPG</title>
    <style>
        body {
            font-family: 'Segoe UI', sans-serif;
            background: #111;
            color: #eee;
            margin: 0;
            padding: 20px;
        }
        h1, h2 {
            text-align: center;
            color: #ffd700;
        }
        .explorador-card {
            border: 2px solid #444;
            padding: 10px;
            margin: 10px;
            width: 180px;
            border-radius: 8px;
            background: #222;
            display: inline-block;
            cursor: pointer;
        }
        .explorador-card.selected {
            border-color: #0f0;
            background: #1a1;
        }
        .boton {
            padding: 10px 20px;
            background-color: #0077cc;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            margin: 10px auto;
            display: block;
        }
        .boton:hover {
            background-color: #005fa3;
        }
        .resultado {
            background: #333;
            padding: 15px;
            margin-top: 20px;
            border-radius: 5px;
        }
        .barra-salud {
            height: 10px;
            background: red;
            margin-top: 5px;
            border-radius: 3px;
        }
        .zona-seleccion {
            margin: 20px auto;
            text-align: center;
        }
        .historial {
            margin-top: 20px;
            font-size: 0.9em;
            background: #222;
            padding: 10px;
            border-radius: 8px;
        }
    </style>
</head>
<body>
    <h1>Gestor de Expediciones</h1>
    <div id="exploradores-container"></div>

    <div class="zona-seleccion">
        <button class="boton" onclick="iniciarExpedicion()">Iniciar Expedición</button>
        <button class="boton" onclick="restaurarSalud()">Campamento (Restaurar Salud)</button>
    </div>

    <div id="resultado" class="resultado"></div>
    <div id="historial" class="historial"></div>

    <script>
        class Explorador {
            constructor(nombre, habilidad, salud = 100) {
                this.nombre = nombre;
                this.habilidad = habilidad;
                this.salud = salud;
            }
            toDict() {
                return {
                    nombre: this.nombre,
                    habilidad: this.habilidad,
                    salud: this.salud
                };
            }
        }

        const exploradores = JSON.parse(localStorage.getItem('exploradores')) || [
            new Explorador("Kael", 85),
            new Explorador("Vira", 60),
            new Explorador("Torn", 45),
            new Explorador("Mira", 70),
            new Explorador("Lyra", 90),
            new Explorador("Brom", 40)
        ].map(e => Object.assign(new Explorador(), e));

        const historial = JSON.parse(localStorage.getItem('historial')) || [];
        const seleccionados = new Set();

        function renderExploradores() {
            const container = document.getElementById('exploradores-container');
            container.innerHTML = '';
            exploradores.forEach((e, idx) => {
                const card = document.createElement('div');
                card.className = 'explorador-card';
                if (seleccionados.has(idx)) card.classList.add('selected');
                card.innerHTML = `
                    <strong>${e.nombre}</strong><br>
                    Habilidad: ${e.habilidad}<br>
                    Salud: ${e.salud}
                    <div class="barra-salud" style="width:${e.salud}%"></div>
                `;
                card.onclick = () => {
                    if (seleccionados.has(idx)) {
                        seleccionados.delete(idx);
                    } else {
                        seleccionados.add(idx);
                    }
                    renderExploradores();
                };
                container.appendChild(card);
            });
        }

        function iniciarExpedicion() {
            if (seleccionados.size === 0) {
                alert("Selecciona al menos un explorador.");
                return;
            }

            const dificultad = 180;
            let totalHabilidad = 0;
            let heridos = [];
            let resultadoTexto = "";

            seleccionados.forEach(idx => {
                totalHabilidad += exploradores[idx].habilidad;
            });

            if (totalHabilidad >= dificultad) {
                const recursos = Math.floor(Math.random() * 80) + 20;
                resultadoTexto = `<h2>¡Éxito!</h2><p>Recursos obtenidos: ${recursos}</p>`;
                historial.push(`✔️ Éxito - Recursos: ${recursos}`);
            } else {
                resultadoTexto = `<h2>Fracaso</h2><ul>`;
                seleccionados.forEach(idx => {
                    const daño = Math.floor(Math.random() * 40) + 10;
                    exploradores[idx].salud = Math.max(0, exploradores[idx].salud - daño);
                    resultadoTexto += `<li>${exploradores[idx].nombre} perdió ${daño} de salud.</li>`;
                });
                resultadoTexto += `</ul>`;
                historial.push(`❌ Fracaso - Habilidad: ${totalHabilidad}`);
            }

            localStorage.setItem('exploradores', JSON.stringify(exploradores));
            localStorage.setItem('historial', JSON.stringify(historial));
            seleccionados.clear();
            document.getElementById('resultado').innerHTML = resultadoTexto;
            renderExploradores();
            mostrarHistorial();
        }

        function restaurarSalud() {
            exploradores.forEach(e => e.salud = 100);
            localStorage.setItem('exploradores', JSON.stringify(exploradores));
            renderExploradores();
            document.getElementById('resultado').innerHTML = "<p>Todos los exploradores han descansado.</p>";
        }

        function mostrarHistorial() {
            document.getElementById('historial').innerHTML = "<h3>Historial de Expediciones</h3>" + historial.map(h => `<p>${h}</p>`).join("");
        }

        renderExploradores();
        mostrarHistorial();
    </script>
</body>
</html>
