<script>
	let number = $state(1);
	let rolling = $state(false);
	let started = $state(false);

	let beta = 0;
	let gamma = 0;
	let lastX = null, lastY = null, lastZ = null;
	let stopTimer, animTimer;

	// Welche Punkte (1 bis 9 im 3x3 Raster) bei jeder Zahl sichtbar sind
	const pips = {
		1: [5],
		2: [1, 9],
		3: [1, 5, 9],
		4: [1, 3, 7, 9],
		5: [1, 3, 5, 7, 9],
		6: [1, 3, 4, 6, 7, 9]
	};

	// Zahl je nach Haltung vom Handy
	function numberFromPosition() {
		if (Math.abs(gamma) > 60) return gamma > 0 ? 3 : 4; // nach rechts / links gekippt
		if (beta > 45 && beta < 135) return 2;              // hochkant
		if (beta < -45 && beta > -135) return 5;            // kopfueber
		if (Math.abs(beta) >= 135) return 6;                // Bildschirm nach unten
		return 1;                                           // flach auf dem Tisch
	}

	function onOrientation(e) {
		beta = e.beta ?? 0;
		gamma = e.gamma ?? 0;
	}

	// Schuetteln erkennen
	function onMotion(e) {
		const a = e.accelerationIncludingGravity;
		if (!a || a.x === null) return;
		if (lastX !== null) {
			const change = Math.abs(a.x - lastX) + Math.abs(a.y - lastY) + Math.abs(a.z - lastZ);
			if (change > 25) shake();
		}
		lastX = a.x;
		lastY = a.y;
		lastZ = a.z;
	}

	function shake() {
		if (!rolling) {
			rolling = true;
			animTimer = setInterval(() => {
				number = Math.floor(Math.random() * 6) + 1;
			}, 100);
		}
		clearTimeout(stopTimer);
		stopTimer = setTimeout(stop, 800);
	}

	function stop() {
		clearInterval(animTimer);
		rolling = false;
		number = numberFromPosition();
	}

	async function start() {
		try {
			// iPhone braucht Erlaubnis fuer beide Sensoren
			if (typeof DeviceMotionEvent?.requestPermission === 'function') {
				const [motion, orientation] = await Promise.all([
					DeviceMotionEvent.requestPermission(),
					DeviceOrientationEvent.requestPermission()
				]);
				if (motion !== 'granted' || orientation !== 'granted') return;
			}
			window.addEventListener('devicemotion', onMotion);
			window.addEventListener('deviceorientation', onOrientation);
			started = true;
		} catch (err) {
			alert('Fehler: ' + err.message);
		}
	}
</script>

<main>
	<h1>Würfel</h1>

	{#if !started}
		<button onclick={start}>Würfel starten</button>
	{:else}
		<div class="dice" class:rolling>
			{#each Array(9) as _, i}
				<div class="cell">
					{#if pips[number].includes(i + 1)}
						<span class="pip"></span>
					{/if}
				</div>
			{/each}
		</div>

		<p class="hint">
			{rolling ? 'Würfelt...' : 'Schüttel das Handy für eine neue Zahl'}
		</p>
	{/if}

	<ul class="legend">
		<li><b>1</b> flach, Bildschirm oben</li>
		<li><b>2</b> hochkant</li>
		<li><b>3</b> nach rechts gekippt</li>
		<li><b>4</b> nach links gekippt</li>
		<li><b>5</b> kopfüber</li>
		<li><b>6</b> Bildschirm nach unten</li>
	</ul>
</main>

<style>
	main {
		min-height: 100vh;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 24px;
		padding: 16px;
		background: linear-gradient(160deg, #1e1b4b, #4c1d95);
		color: white;
		font-family: system-ui, sans-serif;
		text-align: center;
	}

	h1 {
		margin: 0;
		font-size: 2rem;
	}

	button {
		padding: 16px 32px;
		font-size: 1.2rem;
		font-weight: 600;
		border: none;
		border-radius: 999px;
		background: white;
		color: #4c1d95;
		cursor: pointer;
	}

	.dice {
		width: 180px;
		height: 180px;
		padding: 20px;
		box-sizing: border-box;
		background: white;
		border-radius: 28px;
		box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
		display: grid;
		grid-template-columns: repeat(3, 1fr);
		grid-template-rows: repeat(3, 1fr);
	}

	.cell {
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.pip {
		width: 30px;
		height: 30px;
		border-radius: 50%;
		background: #1e1b4b;
	}

	.rolling {
		animation: wobble 0.3s infinite;
	}

	@keyframes wobble {
		0% { transform: rotate(0deg) scale(1); }
		25% { transform: rotate(-12deg) scale(0.95); }
		75% { transform: rotate(12deg) scale(0.95); }
		100% { transform: rotate(0deg) scale(1); }
	}

	.hint {
		margin: 0;
		opacity: 0.85;
	}

	.legend {
		list-style: none;
		padding: 0;
		margin: 0;
		font-size: 0.9rem;
		opacity: 0.7;
		line-height: 1.7;
	}
</style>