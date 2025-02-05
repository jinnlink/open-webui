<script>
	import { onMount } from 'svelte';
	import { goto } from '$app/navigation';
	import { user } from '$lib/stores';

	let container;
	let dogContainer;
	let celebrationText;
	let redirectTimeout;
	
	function createConfetti() {
		const confetti = document.createElement('div');
		confetti.className = 'confetti';
		confetti.style.left = Math.random() * 100 + 'vw';
		confetti.style.animationDelay = Math.random() * 3 + 's';
		confetti.style.backgroundColor = `hsl(${Math.random() * 360}deg, 100%, 50%)`;
		container.appendChild(confetti);
		
		// 在动画结束后移除彩带元素
		confetti.addEventListener('animationend', () => {
			confetti.remove();
		});
	}
	
	onMount(() => {
		// 如果用户已登录，直接跳转到主页
		if ($user) {
			goto('/chat');
			return;
		}

		// 创建多个彩带
		for (let i = 0; i < 50; i++) {
			createConfetti();
		}

		// 每隔一段时间创建新的彩带
		const interval = setInterval(() => {
			createConfetti();
		}, 300);

		// 15秒后跳转到登录页
		redirectTimeout = setTimeout(() => {
			clearInterval(interval);
			goto('/auth');
		}, 15000);

		// 小狗奔跑动画
		const dog = document.createElement('div');
		dog.className = 'running-dog';
		dogContainer.appendChild(dog);

		// 文字动画
		celebrationText.classList.add('text-animation');

		return () => {
			clearInterval(interval);
			clearTimeout(redirectTimeout);
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
		background: linear-gradient(135deg, #ff6b6b, #ffd93d, #6c5ce7);
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		overflow: hidden;
		z-index: 1000;
	}

	.text-container {
		text-align: center;
		opacity: 0;
		z-index: 2;
	}

	.text-container h1 {
		font-size: 4rem;
		color: #fff;
		text-shadow: 0 0 10px rgba(0,0,0,0.3);
		margin: 0;
		padding: 20px;
		background: linear-gradient(45deg, #ff0, #f0f, #0ff);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		animation: rainbow 3s ease-in-out infinite;
	}

	.dog-container {
		margin-top: 50px;
		height: 100px;
		width: 100%;
		position: relative;
		z-index: 2;
	}

	.running-dog {
		width: 100px;
		height: 100px;
		background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'%3E%3Cpath d='M20,60 C30,40 40,40 50,60 C60,80 70,80 80,60' stroke='white' stroke-width='8' fill='none' stroke-linecap='round'/%3E%3Ccircle cx='35' cy='45' r='5' fill='white'/%3E%3Ccircle cx='65' cy='45' r='5' fill='white'/%3E%3C/svg%3E");
		position: absolute;
		left: -100px;
		animation: runDog 4s linear infinite;
	}

	.confetti {
		position: absolute;
		width: 10px;
		height: 20px;
		animation: fall 3s linear infinite;
	}

	@keyframes fall {
		0% {
			transform: translateY(-100vh) rotate(0deg);
			opacity: 1;
		}
		100% {
			transform: translateY(100vh) rotate(360deg);
			opacity: 0;
		}
	}

	@keyframes rainbow {
		0% { filter: hue-rotate(0deg); }
		100% { filter: hue-rotate(360deg); }
	}

	@keyframes runDog {
		0% {
			left: -100px;
			transform: scaleX(1);
		}
		45% {
			left: calc(100% + 50px);
			transform: scaleX(1);
		}
		50% {
			left: calc(100% + 50px);
			transform: scaleX(-1);
		}
		95% {
			left: -100px;
			transform: scaleX(-1);
		}
		100% {
			left: -100px;
			transform: scaleX(1);
		}
	}

	.text-animation {
		animation: fadeInUp 1s ease forwards;
	}

	@keyframes fadeInUp {
		from {
			opacity: 0;
			transform: translateY(20px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	/* 响应式设计 */
	@media (max-width: 768px) {
		.text-container h1 {
			font-size: 2rem;
			padding: 10px;
		}
		
		.running-dog {
			width: 60px;
			height: 60px;
		}
	}
</style>
