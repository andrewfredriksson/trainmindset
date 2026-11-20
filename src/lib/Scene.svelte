<script lang="ts">
	import { onMount } from 'svelte';

	let {
		order = 0,
		holding = $bindable(false),
		held = $bindable(0),
		phase = $bindable('')
	}: { order?: number; holding?: boolean; held?: number; phase?: string } = $props();

	let canvas: HTMLCanvasElement;

	const vert = /* glsl */ `
		uniform float uTime;
		uniform float uOrder;
		uniform float uBreath;
		uniform float uPx;
		attribute vec3 aOrder;
		attribute float aSeed;
		varying float vAlpha;
		varying float vAccent;

		void main() {
			float t = uTime * 0.18;
			vec3 chaos = position;
			chaos += vec3(
				sin(t * 3.1 + aSeed * 41.0 + position.y * 0.8),
				cos(t * 2.7 + aSeed * 33.0 + position.z * 0.8),
				sin(t * 2.3 + aSeed * 27.0 + position.x * 0.8)
			) * (0.5 + uOrder * 0.0) * 1.6;

			float r = length(aOrder.xz);
			vec3 calm = aOrder * (1.0 + 0.07 * uBreath);
			calm.y += sin(r * 7.0 - uTime * 0.7) * 0.025 * (0.4 + uBreath);

			float k = smoothstep(aSeed * 0.55, aSeed * 0.55 + 0.45, uOrder);
			vec3 p = mix(chaos, calm, k);

			vec4 mv = modelViewMatrix * vec4(p, 1.0);
			gl_Position = projectionMatrix * mv;
			gl_PointSize = uPx * (1.6 + aSeed * 2.4) * (7.5 / -mv.z);

			vAlpha = mix(0.28, 0.95, k);
			vAccent = step(0.975, aSeed);
		}
	`;

	const frag = /* glsl */ `
		precision mediump float;
		varying float vAlpha;
		varying float vAccent;
		void main() {
			float d = length(gl_PointCoord - 0.5);
			if (d > 0.5) discard;
			float a = smoothstep(0.5, 0.05, d);
			vec3 bone = vec3(0.925, 0.902, 0.847);
			vec3 ember = vec3(1.0, 0.357, 0.122);
			gl_FragColor = vec4(mix(bone, ember, vAccent), a * vAlpha);
		}
	`;

	onMount(() => {
		let disposed = false;
		let raf = 0;
		let cleanup = () => {};

		(async () => {
			const THREE = await import('three');
			if (disposed) return;

			let renderer: InstanceType<typeof THREE.WebGLRenderer>;
			try {
				renderer = new THREE.WebGLRenderer({ canvas, alpha: true, antialias: false });
			} catch {
				return;
			}
			const reduced = matchMedia('(prefers-reduced-motion: reduce)').matches;
			const small = innerWidth < 760;
			const px = Math.min(devicePixelRatio, 2);
			renderer.setPixelRatio(px);
			renderer.setClearColor(0x000000, 0);

			const scene = new THREE.Scene();
			const camera = new THREE.PerspectiveCamera(38, 1, 0.1, 60);
			camera.position.set(0, 3.4, 8.2);
			camera.lookAt(0, 0, 0);

			const N = small ? 9000 : 24000;
			const RINGS = 28;
			const pos = new Float32Array(N * 3);
			const ord = new Float32Array(N * 3);
			const seed = new Float32Array(N);
			for (let i = 0; i < N; i++) {
				// noisy cloud: gaussian-ish volume
				const g = () => (Math.random() + Math.random() + Math.random() - 1.5) * 2;
				pos[i * 3] = g() * 4.2;
				pos[i * 3 + 1] = g() * 2.6;
				pos[i * 3 + 2] = g() * 4.2;
				// still water: concentric rings
				const ring = Math.floor(Math.random() * RINGS);
				const radius = 0.5 + ring * 0.115 + (Math.random() - 0.5) * 0.012;
				const ang = Math.random() * Math.PI * 2;
				ord[i * 3] = Math.cos(ang) * radius;
				ord[i * 3 + 1] = 0;
				ord[i * 3 + 2] = Math.sin(ang) * radius;
				seed[i] = Math.random();
			}
			const geo = new THREE.BufferGeometry();
			geo.setAttribute('position', new THREE.BufferAttribute(pos, 3));
			geo.setAttribute('aOrder', new THREE.BufferAttribute(ord, 3));
			geo.setAttribute('aSeed', new THREE.BufferAttribute(seed, 1));

			const mat = new THREE.ShaderMaterial({
				vertexShader: vert,
				fragmentShader: frag,
				transparent: true,
				depthWrite: false,
				uniforms: {
					uTime: { value: 0 },
					uOrder: { value: 0 },
					uBreath: { value: 0 },
					uPx: { value: px }
				}
			});
			const points = new THREE.Points(geo, mat);
			scene.add(points);

			const resize = () => {
				const w = innerWidth;
				const h = innerHeight;
				renderer.setSize(w, h, false);
				camera.aspect = w / h;
				camera.position.z = w < 760 ? 11 : 8.2;
				camera.updateProjectionMatrix();
			};
			resize();
			addEventListener('resize', resize);

			const ptr = { x: 0, y: 0 };
			const onMove = (e: PointerEvent) => {
				ptr.x = (e.clientX / innerWidth - 0.5) * 2;
				ptr.y = (e.clientY / innerHeight - 0.5) * 2;
			};
			const onDown = (e: PointerEvent) => {
				if ((e.target as HTMLElement).closest('a,button,input,textarea')) return;
				holding = true;
			};
			const onUp = () => (holding = false);
			addEventListener('pointermove', onMove);
			addEventListener('pointerdown', onDown);
			addEventListener('pointerup', onUp);
			addEventListener('pointercancel', onUp);
			addEventListener('blur', onUp);

			let hold = 0; // 0..1 eased
			let heldT = 0; // seconds in current hold
			let last = performance.now();
			let time = 0;
			let lastHeldInt = -1;
			let lastPhase = '';

			const tick = (now: number) => {
				raf = requestAnimationFrame(tick);
				const dt = Math.min((now - last) / 1000, 0.05);
				last = now;
				time += dt * (reduced ? 0.25 : 1);

				if (holding) {
					heldT += dt;
					hold += (1 - hold) * (1 - Math.exp(-dt * 0.9));
				} else {
					heldT = 0;
					hold += (0 - hold) * (1 - Math.exp(-dt * 1.4));
				}

				// 4s in, 6s out
				let breath: number;
				let ph = '';
				if (holding) {
					const c = heldT % 10;
					if (c < 4) {
						breath = c / 4;
						ph = 'in';
					} else {
						breath = 1 - (c - 4) / 6;
						ph = 'out';
					}
					breath = breath * breath * (3 - 2 * breath);
				} else {
					breath = 0.5 + 0.5 * Math.sin(time * 0.5);
				}

				const target = Math.max(order, hold);
				const u = mat.uniforms;
				u.uTime.value = time;
				u.uOrder.value += (target - u.uOrder.value) * (1 - Math.exp(-dt * 2.2));
				u.uBreath.value += (breath - u.uBreath.value) * (1 - Math.exp(-dt * 4));

				points.rotation.y += dt * 0.05 * (1 - u.uOrder.value * 0.8);
				points.rotation.x += (ptr.y * 0.08 - points.rotation.x) * 0.04;
				camera.position.x += (ptr.x * 0.6 - camera.position.x) * 0.04;
				camera.lookAt(0, 0, 0);

				const hi = Math.floor(heldT);
				if (hi !== lastHeldInt) {
					lastHeldInt = hi;
					held = hi;
				}
				if (ph !== lastPhase) {
					lastPhase = ph;
					phase = ph;
				}

				renderer.render(scene, camera);
			};
			raf = requestAnimationFrame(tick);

			cleanup = () => {
				removeEventListener('resize', resize);
				removeEventListener('pointermove', onMove);
				removeEventListener('pointerdown', onDown);
				removeEventListener('pointerup', onUp);
				removeEventListener('pointercancel', onUp);
				removeEventListener('blur', onUp);
				geo.dispose();
				mat.dispose();
				renderer.dispose();
			};
		})();

		return () => {
			disposed = true;
			cancelAnimationFrame(raf);
			cleanup();
		};
	});
</script>

<canvas bind:this={canvas} aria-hidden="true" class="pointer-events-none fixed inset-0 z-0 h-full w-full"
></canvas>
