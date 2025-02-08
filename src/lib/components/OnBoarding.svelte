<script>
	import { getContext } from 'svelte';
	const i18n = getContext('i18n');

	import { WEBUI_BASE_URL } from '$lib/constants';

	import Marquee from './common/Marquee.svelte';
	import SlideShow from './common/SlideShow.svelte';
	import ArrowRightCircle from './icons/ArrowRightCircle.svelte';

	export let show = true;
	export let getStartedHandler = () => {};
</script>

{#if show}
	<div class="w-full h-screen max-h-[100dvh] text-white relative">
		<div class="fixed m-10 z-50">
			<div class="flex space-x-2">
				<div class="self-center logo-glow">
					<img
						crossorigin="anonymous"
						src="{WEBUI_BASE_URL}/static/splash.png"
						class="w-16 rounded-xl dark:invert hover:scale-110 transition-all duration-500 ease-out"
						alt="logo"
					/>
				</div>
			</div>
		</div>

		<SlideShow duration={5000} />

		<div
			class="w-full h-full absolute top-0 left-0 bg-gradient-to-t from-20% from-black/70 to-transparent"
		></div>

		<div class="w-full h-full absolute top-0 left-0 backdrop-blur-[2px] bg-black/20"></div>

		<div class="relative bg-transparent w-full min-h-screen flex z-10">
			<div class="flex flex-col justify-end w-full items-center pb-10 text-center">
				<div class="text-5xl lg:text-7xl font-secondary mystic-text slide-text">
					<div class="marquee-container">
						<div class="bg-gradient-to-r from-cyan-100 via-teal-100 to-emerald-100 bg-clip-text text-transparent animate-gradient">
							<Marquee
								duration={5000}
								words={[
									$i18n.t('Explore the cosmos'),
									$i18n.t('Unlock mysteries'),
									$i18n.t('Chart new frontiers'),
									$i18n.t('Dive into knowledge'),
									$i18n.t('Discover wonders'),
									$i18n.t('Ignite curiosity'),
									$i18n.t('Forge new paths'),
									$i18n.t('Unravel secrets'),
									$i18n.t('Pioneer insights'),
									$i18n.t('Embark on adventures')
								]}
							/>
						</div>
					</div>
					<div class="mt-4 text-lg text-gray-200/90 mystic-text opacity-90">{$i18n.t(`wherever you are`)}</div>
				</div>

				<div class="flex justify-center mt-12">
					<div class="flex flex-col justify-center items-center">
						<button
							class="relative z-20 flex p-4 rounded-xl btn-mystic group"
							on:click={() => {
								getStartedHandler();
							}}
						>
							<ArrowRightCircle className="size-6 group-hover:translate-x-1 transition-transform duration-300" />
						</button>
						<div class="mt-4 font-primary text-lg font-medium mystic-text text-transparent bg-clip-text bg-gradient-to-r from-cyan-100 via-teal-100 to-emerald-100 animate-gradient">
							{$i18n.t(`Get started`)}
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
{/if}

<svelte:head>
	<link href="https://fonts.googleapis.com/css2?family=ZCOOL+XiaoWei&family=Noto+Serif+SC:wght@400;500&display=swap" rel="stylesheet">
</svelte:head>

<style>
	.animate-gradient {
		background-size: 200% auto;
		animation: gradient 8s linear infinite;
		transition: all 0.3s ease;
	}
	
	@keyframes gradient {
		0% { background-position: 0% 50%; }
		50% { background-position: 100% 50%; }
		100% { background-position: 0% 50%; }
	}
	
	.mystic-text {
		font-family: 'ZCOOL XiaoWei', 'Noto Serif SC', '思源宋体', 'Source Han Serif SC', serif;
		background: linear-gradient(45deg, #ff6b6b, #4ecdc4, #45b7d1, #96f2d7);
		background-size: 400% 400%;
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		text-shadow: 0 0 20px rgba(78, 205, 196, 0.8), 0 0 40px rgba(78, 205, 196, 0.6), 0 0 60px rgba(78, 205, 196, 0.4);
		animation: text-gradient 12s ease infinite, text-glow 2s ease-in-out infinite alternate;
	}

	@keyframes text-gradient {
		0% { background-position: 0% 50%; }
		50% { background-position: 100% 50%; }
		100% { background-position: 0% 50%; }
	}

	@keyframes text-glow {
		from {
			filter: hue-rotate(0deg);
			text-shadow: 0 0 10px rgba(0, 255, 255, 0.8), 0 0 20px rgba(255, 0, 255, 0.6), 0 0 30px rgba(0, 255, 0, 0.4);
		}
		to {
			filter: hue-rotate(360deg);
			text-shadow: 0 0 20px rgba(0, 255, 255, 1), 0 0 40px rgba(255, 0, 255, 0.8), 0 0 60px rgba(0, 255, 0, 0.6);
		}
	}

	.btn-mystic {
		background: linear-gradient(135deg, rgba(141, 255, 249, 0.1), rgba(141, 255, 249, 0.15));
		border: 1px solid rgba(141, 255, 249, 0.2);
		color: rgba(255, 255, 255, 0.95);
		font-family: 'Noto Serif SC', '思源宋体', serif;
		letter-spacing: 0.1em;
		transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
		backdrop-filter: blur(5px);
	}

	.btn-mystic:hover {
		background: linear-gradient(135deg, rgba(141, 255, 249, 0.15), rgba(141, 255, 249, 0.2));
		border-color: rgba(141, 255, 249, 0.4);
		box-shadow: 0 0 20px rgba(141, 255, 249, 0.3);
		transform: translateY(-2px) scale(1.02);
	}

	.slide-text {
		animation: slideUp 0.6s cubic-bezier(0.4, 0, 0.2, 1) forwards;
		opacity: 0;
		transform: translateY(20px);
	}

	@keyframes slideUp {
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	.logo-glow {
		position: relative;
	}

	.logo-glow::after {
		content: '';
		position: absolute;
		inset: -10px;
		background: radial-gradient(circle, rgba(141, 255, 249, 0.2) 0%, transparent 70%);
		z-index: -1;
		opacity: 0;
		transition: opacity 0.3s ease;
	}

	.logo-glow:hover::after {
		opacity: 1;
	}

	.marquee-container {
		position: relative;
		overflow: hidden;
	}

	.marquee-container::before,
	.marquee-container::after {
		content: '';
		position: absolute;
		top: 0;
		bottom: 0;
		width: 50px;
		z-index: 2;
		pointer-events: none;
	}

	.marquee-container::before {
		left: 0;
		background: linear-gradient(to right, rgba(0, 0, 0, 0.8), transparent);
	}

	.marquee-container::after {
		right: 0;
		background: linear-gradient(to left, rgba(0, 0, 0, 0.8), transparent);
	}
</style>
