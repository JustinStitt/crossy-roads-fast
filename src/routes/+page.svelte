<script>
	import P5 from 'p5-svelte';

	let cwidth = 1000;
	let cheight = 600;

	let grid_size = 50; // 50x50 px across

	let win = false;
	let lose = false;

	const player = {
		r: cheight / grid_size / 2,
		c: 1,
		want_c: 1,
		render_c: 1 * grid_size,
		want_r: cheight / grid_size / 2,
		render_r: cheight / 2,
		radius: 25
	};

	const draw_player = (p5) => {
		// lerp player pos
		player.render_c = lerp(player.render_c, player.want_c * grid_size, 0.3);
		player.render_r = lerp(player.render_r, player.want_r * grid_size, 0.3);
		if (Math.abs(player.want_c * grid_size - player.render_c) < 5) {
			player.c = player.want_c;
			player.render_c = player.c * grid_size;
		}
		if (Math.abs(player.want_r * grid_size - player.render_r) < 5) {
			player.r = player.want_r;
			player.render_r = player.r * grid_size;
		}

		p5.ellipse(player.render_c, player.render_r, player.radius, player.radius);
		// console.log(player.c, player.want_c, player.render_c);
	};

	const handleRight = () => {
		// player.c += 1;
		if (player.c == player.want_c) player.want_c += 1;
		if (player.c == cwidth / grid_size) {
			triggerWinSequence();
		}
		// player.c = Math.min(Math.max(player.c, 0), cwidth / grid_size);

		// move all enemies to the left to simulate moving forward
		// enemies.forEach((e) => {
		// 	e.x -= grid_size;
		// });
	};

	function lerp(a, b, alpha) {
		return a + alpha * (b - a);
	}

	const handleUp = () => {
		if (player.r == player.want_r) {
			player.want_r -= 1;
			player.r = Math.min(Math.max(player.r, 0), cheight / grid_size);
		}
	};

	const handleDown = () => {
		if (player.r == player.want_r) {
			player.want_r += 1;
			player.r = Math.min(Math.max(player.r, 0), cheight / grid_size);
		}
	};

	const triggerWinSequence = (p5) => {
		win = true;
	};

	const triggerLoseSequence = (p5) => {
		if (win || lose) return;
		console.log('you lose');
		lose = true;
		player.r = -5;
		player.c = -5;
		window.alert('YOU LOSE HAHA');
	};

	class Enemy {
		constructor(x, y, speed, w, h) {
			this.x = x;
			this.y = y;
			this.speed = speed;
			this.w = w;
			this.h = h;
		}

		self_destroy() {
			const index = enemies.indexOf(this);
			if (index > -1) {
				// only splice array when item is found
				enemies.splice(index, 1); // 2nd parameter means remove one item only
			}
		}

		draw(p5) {
			p5.stroke(200, 40, 11);

			//update position
			this.y += this.speed;
			p5.rect(this.x, this.y, this.w, this.h);

			// collision
			// check if center of circle is in this bounding box
			const pcx = player.c * grid_size;
			const pcy = player.r * grid_size;
			const wf = this.x + this.w;
			const hf = this.y + this.h;
			if (pcx > this.x && pcx < wf && pcy > this.y && pcy < hf) {
				console.log('you lose');
				triggerLoseSequence();
			}

			// check self destroy
			if (this.speed > 0 && this.y > cheight + 10) this.self_destroy();
			else if (this.speed < 0 && this.y < -10) this.self_destroy();
		}
	}

	const enemies = [];

	const spawnEnemy = () => {
		let rx = Math.floor(Math.random() * (cwidth / grid_size) + 2);
		// top or bottom for ry
		let dir = 1;
		let ry = -2;
		if (Math.random() < 0.5) {
			dir = -1;
			ry = cheight / grid_size + 3;
		}
		let random_speed = Math.floor(Math.random() * 10) + 2;

		const new_enemy = new Enemy(
			rx * grid_size + grid_size / 2,
			ry * grid_size,
			// 750,
			// 0,
			// dir * random_speed * 0,
			dir * random_speed,
			grid_size,
			grid_size * 2
		); // x, y, speed, w, h
		enemies.unshift(new_enemy);
	};

	spawnEnemy();
	setInterval(() => {
		if (!lose && !win) spawnEnemy();
	}, 200);

	const sketch = (p5) => {
		p5.setup = () => {
			p5.createCanvas(cwidth, cheight);
		};

		p5.draw = () => {
			p5.background(42, 42, 42);
			p5.stroke(0, 99, 44);
			p5.strokeWeight(4);
			p5.textSize(200);
			draw_player(p5);
			// p5.ellipse(p5.width / 2, p5.height / 2, 50, 50);
			enemies.forEach((e) => {
				e.draw(p5);
			});
			if (win) {
				p5.text('YOU WIN!!!!', 50, cheight / 2);
			} else if (lose) {
				p5.text('YOU LOSE!!!!', 50, cheight / 2);
			}
		};

		p5.keyPressed = (key) => {
			if (key.code == 'Space' || key.code == 'ArrowRight') {
				handleRight();
			} else if (key.code == 'ArrowUp') {
				handleUp();
			} else if (key.code == 'ArrowDown') {
				handleDown();
			}
		};
	};
</script>

<div class="main">
	<h1>Crossy Roads but sideways lol</h1>
	<P5 {sketch} />
</div>

<style>
	.main {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}
</style>
