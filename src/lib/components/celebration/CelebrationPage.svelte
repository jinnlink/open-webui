<script>
	import { onMount } from 'svelte';
	import { user } from '$lib/stores';

	let container;
	let dogContainer;
	let celebrationText;
	let confettiInterval;

	function createConfetti() {
		try {
			const confetti = document.createElement('div');
			confetti.className = 'confetti';
			confetti.style.left = Math.random() * 100 + 'vw';
			confetti.style.animationDelay = Math.random() * 3 + 's';
			confetti.style.backgroundColor = `hsl(${Math.random() * 360}deg, 100%, 50%)`;
			container?.appendChild(confetti);
			
			// 在动画结束后移除彩带元素
			confetti.addEventListener('animationend', () => {
				confetti?.remove();
			});
		} catch (error) {
			console.error('Error creating confetti:', error);
		}
	}
	
	onMount(async () => {
		try {
			// 创建多个彩带
			for (let i = 0; i < 30; i++) {
				createConfetti();
			}

			// 每隔一段时间创建新的彩带
			confettiInterval = setInterval(() => {
				createConfetti();
			}, 500);

			// 小狗奔跑动画
			if (dogContainer) {
				const dog = document.createElement('div');
				dog.className = 'running-dog';
				dogContainer.appendChild(dog);
			}

			// 文字动画
			if (celebrationText) {
				celebrationText.classList.add('text-animation');
			}
		} catch (error) {
			console.error('Error in celebration setup:', error);
		}

		return () => {
			if (confettiInterval) clearInterval(confettiInterval);
		};
	});
</script>

<div class="celebration-container" bind:this={container}>
	<div class="text-container" bind:this={celebrationText}>
		<h1>热烈庆祝翔宝贝诞辰</h1>
	</div>
	<div class="dog-container" bind:this={dogContainer}></div>
</div>

<style>
	.celebration-container {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		overflow: hidden;
		z-index: 20;
		pointer-events: none;
	}

	.text-container {
		position: absolute;
		top: 20%;
		text-align: center;
		opacity: 0;
	}

	.text-container h1 {
		font-size: 2.5rem;
		color: #ffd700;
		text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
		margin: 0;
	}

	.text-animation {
		animation: fadeInOut 4s ease-in-out infinite;
	}

	.confetti {
		position: fixed;
		width: 10px;
		height: 10px;
		background-color: #f00;
		pointer-events: none;
		opacity: 0.8;
		animation: fall 3s linear forwards;
	}

	.dog-container {
		position: absolute;
		bottom: 20%;
		width: 100%;
		height: 100px;
		overflow: hidden;
	}

	.running-dog {
		position: absolute;
		width: 50px;
		height: 50px;
		background-image: url('/static/dog.gif');
		background-size: contain;
		background-repeat: no-repeat;
		animation: runAcross 8s linear infinite;
	}

	@keyframes fall {
		0% {
			transform: translateY(-100vh) rotate(0deg);
		}
		100% {
			transform: translateY(100vh) rotate(360deg);
		}
	}

	@keyframes fadeInOut {
		0%, 100% {
			opacity: 0;
			transform: translateY(-20px);
		}
		50% {
			opacity: 1;
			transform: translateY(0);
		}
	}

	@keyframes runAcross {
		0% {
			left: -50px;
		}
		100% {
			left: calc(100% + 50px);
		}
	}
</style>
