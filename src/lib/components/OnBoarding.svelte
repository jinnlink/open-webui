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
				<div class=" self-center">
					<img
						crossorigin="anonymous"
						src="{WEBUI_BASE_URL}/static/splash.png"
						class="w-16 rounded-xl dark:invert hover:scale-110 transition-transform animate-pulse"
						alt="logo"
					/>
				</div>
			</div>
		</div>

		<SlideShow duration={5000} />

		<div
			class="w-full h-full absolute top-0 left-0 bg-gradient-to-t from-20% from-black/60 to-transparent"
		></div>

		<div class="w-full h-full absolute top-0 left-0 backdrop-blur-sm bg-black/30"></div>

		<div class="relative bg-transparent w-full min-h-screen flex z-10">
			<div class="flex flex-col justify-end w-full items-center pb-10 text-center">
				<div class="text-5xl lg:text-7xl font-secondary mystic-text">
					<div class="bg-gradient-to-r from-teal-200 via-emerald-200 to-cyan-200 bg-clip-text text-transparent animate-gradient">
						<Marquee
							duration={5000}
							words={[
								$i18n.t('热烈祝贺翔宝贝诞辰'),
								$i18n.t('特别纪念翔宝贝诞生'),
								$i18n.t('拉莱耶领袖克苏鲁特别贺梦'),
								$i18n.t('特邀前往梦中之城庆祝'),
								$i18n.t('旧日支配者为您献上祝福'),
								$i18n.t('红帽小狗西昂已做好准备'),
								$i18n.t('诚邀您前往一同庆祝翔宝贝诞生'),
								$i18n.t('深潜者已在夹道欢呼'),
								$i18n.t('来吧来吧，遵从呼唤，纵情欢庆'),
								$i18n.t('与红帽小狗西昂一起踏上旅途')
							]}
						/>
					</div>
					<div class="mt-2 text-lg text-gray-300 mystic-text">{$i18n.t(`wherever you are`)}</div>
				</div>

				<div class="flex justify-center mt-8">
					<div class="flex flex-col justify-center items-center">
						<button
							class="relative z-20 flex p-3 rounded-xl bg-black/30 hover:bg-black/40 border border-emerald-900/30 hover:border-emerald-700/30 transition-all duration-300 shadow-[0_0_15px_rgba(0,255,255,0.15)] hover:shadow-[0_0_25px_rgba(0,255,255,0.25)]"
							on:click={() => {
								getStartedHandler();
							}}
						>
							<ArrowRightCircle className="size-6" />
						</button>
						<div class="mt-3 font-primary text-lg font-medium mystic-text text-transparent bg-clip-text bg-gradient-to-r from-teal-200 via-emerald-200 to-cyan-200 animate-gradient">
							{$i18n.t(`Get started`)}
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
{/if}

<svelte:head>
	<link href="https://fonts.googleapis.com/css2?family=Ma+Shan+Zheng&display=swap" rel="stylesheet">
</svelte:head>

<style>
	.animate-gradient {
		background-size: 200% auto;
		animation: gradient 8s linear infinite;
		transition: all 0.3s ease;
	}
	
	@keyframes gradient {
		0% {
			background-position: 0% 50%;
		}
		50% {
			background-position: 100% 50%;
		}
		100% {
			background-position: 0% 50%;
		}
	}
	
	.mystic-text {
		font-family: 'Ma Shan Zheng', '华文行楷', STXingkai, '楷体', KaiTi, serif;
		text-shadow: 
			0 0 15px rgba(141, 255, 249, 0.3),
			0 0 30px rgba(141, 255, 249, 0.2);
		letter-spacing: 0.2em;
		font-weight: 400;
		-webkit-text-stroke: 0.5px rgba(141, 255, 249, 0.3);
	}
</style>
