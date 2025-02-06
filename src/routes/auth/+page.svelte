<script>
	import { toast } from 'svelte-sonner';

	import { onMount, getContext } from 'svelte';
	import { goto } from '$app/navigation';
	import { page } from '$app/stores';

	import { getBackendConfig } from '$lib/apis';
	import { ldapUserSignIn, getSessionUser, userSignIn, userSignUp } from '$lib/apis/auths';

	import { WEBUI_API_BASE_URL, WEBUI_BASE_URL } from '$lib/constants';
	import { WEBUI_NAME, config, user, socket } from '$lib/stores';

	import { generateInitialsImage, canvasPixelTest } from '$lib/utils';

	import Spinner from '$lib/components/common/Spinner.svelte';
	import OnBoarding from '$lib/components/OnBoarding.svelte';

	import { fade, fly } from 'svelte/transition';

	const i18n = getContext('i18n');

	let loaded = false;

	let mode = 'signin';

	let name = '';
	let email = '';
	let password = '';

	let ldapUsername = '';

	const setSessionUser = async (sessionUser) => {
		if (sessionUser) {
			console.log(sessionUser);
			toast.success($i18n.t(`You're now logged in.`));
			if (sessionUser.token) {
				localStorage.token = sessionUser.token;
			}

			$socket.emit('user-join', { auth: { token: sessionUser.token } });
			await user.set(sessionUser);
			await config.set(await getBackendConfig());
			goto('/');
		}
	};

	const signInHandler = async () => {
		const sessionUser = await userSignIn(email, password).catch((error) => {
			toast.error(`${error}`);
			return null;
		});

		await setSessionUser(sessionUser);
	};

	const signUpHandler = async () => {
		const sessionUser = await userSignUp(name, email, password, generateInitialsImage(name)).catch(
			(error) => {
				toast.error(`${error}`);
				return null;
			}
		);

		await setSessionUser(sessionUser);
	};

	const ldapSignInHandler = async () => {
		const sessionUser = await ldapUserSignIn(ldapUsername, password).catch((error) => {
			toast.error(`${error}`);
			return null;
		});
		await setSessionUser(sessionUser);
	};

	const submitHandler = async () => {
		if (mode === 'ldap') {
			await ldapSignInHandler();
		} else if (mode === 'signin') {
			await signInHandler();
		} else {
			await signUpHandler();
		}
	};

	const checkOauthCallback = async () => {
		if (!$page.url.hash) {
			return;
		}
		const hash = $page.url.hash.substring(1);
		if (!hash) {
			return;
		}
		const params = new URLSearchParams(hash);
		const token = params.get('token');
		if (!token) {
			return;
		}
		const sessionUser = await getSessionUser(token).catch((error) => {
			toast.error(`${error}`);
			return null;
		});
		if (!sessionUser) {
			return;
		}
		localStorage.token = token;
		await setSessionUser(sessionUser);
	};

	let onboarding = true;

	onMount(async () => {
		try {
			const urlParams = new URLSearchParams(window.location.search);
			const action = urlParams.get('action');
			
			// 只有在URL中明确指定action=signup时才显示注册界面
			if (action === 'signup' && $config?.features.enable_signup) {
				mode = 'signup';
			} else {
				// 其他所有情况都显示普通登录界面
				mode = 'signin';
			}
			
			if ($user !== undefined) {
				await goto('/');
			}
			await checkOauthCallback();

			loaded = true;
			if (($config?.features.auth_trusted_header ?? false) || $config?.features.auth === false) {
				await signInHandler();
			} else {
				onboarding = true;
			}
		} catch (error) {
			console.error('Error in auth page mount:', error);
			loaded = true;
		}
	});
</script>

<svelte:head>
	<title>
		{`${$WEBUI_NAME}`}
	</title>
</svelte:head>

<OnBoarding
	bind:show={onboarding}
	getStartedHandler={() => {
		onboarding = false;
		mode = $config?.features.enable_ldap ? 'ldap' : 'signup';
	}}
/>

<div class="w-full h-screen max-h-[100dvh] text-white relative">
	<div class="w-full h-full absolute top-0 left-0 bg-black dark:bg-black"></div>

	<div class="w-full absolute top-0 left-0 right-0 h-8 drag-region" />

	{#if loaded}
		<div 
			class="fixed left-1/2 top-[15%] -translate-x-1/2 z-50"
			in:fly={{ y: -20, duration: 600, delay: 100 }}
		>
			<div class="flex flex-col items-center space-y-4">
				<div class="relative group">
					<img
						crossorigin="anonymous"
						src="{WEBUI_BASE_URL}/static/splash.png"
						class="w-20 rounded-xl dark:invert hover:scale-110 transition-transform duration-300"
						alt="logo"
					/>
					<div class="absolute inset-0 rounded-xl shadow-[0_0_15px_rgba(0,255,255,0.15)] group-hover:shadow-[0_0_25px_rgba(0,255,255,0.25)] transition-shadow duration-300"></div>
				</div>
				<div class="text-2xl font-medium text-transparent bg-clip-text bg-gradient-to-r from-emerald-400 via-purple-500 to-emerald-400 animate-gradient">
					{$WEBUI_NAME}
				</div>
			</div>
		</div>

		<div
			class="fixed bg-transparent min-h-screen w-full flex justify-center font-primary z-40 text-gray-200"
		>
			<div 
				class="w-full sm:max-w-md px-10 min-h-screen flex flex-col text-center"
				in:fly={{ y: 20, duration: 600, delay: 200 }}
			>
				{#if ($config?.features.auth_trusted_header ?? false) || $config?.features.auth === false}
					<div class="my-auto pb-10 w-full">
						<div
							class="flex items-center justify-center gap-3 text-xl sm:text-2xl text-center font-semibold text-gray-200"
						>
							<div>
								{$i18n.t('Signing in to {{WEBUI_NAME}}', { WEBUI_NAME: $WEBUI_NAME })}
							</div>

							<div>
								<Spinner />
							</div>
						</div>
					</div>
				{:else}
					<div class="my-auto pb-10 w-full">
						<form
							class="flex flex-col justify-center backdrop-blur-lg bg-black/30 p-8 rounded-2xl shadow-[0_0_15px_rgba(0,255,255,0.15)] border border-emerald-900/30 hover:shadow-[0_0_25px_rgba(0,255,255,0.25)] transition-all duration-500"
							on:submit={(e) => {
								e.preventDefault();
								submitHandler();
							}}
						>
							<div class="mb-6">
								<div class="text-2xl font-medium bg-gradient-to-r from-emerald-400 via-purple-500 to-emerald-400 bg-clip-text text-transparent animate-gradient">
									{#if mode === 'ldap'}
										{$i18n.t(`Sign in to {{WEBUI_NAME}} with LDAP`, { WEBUI_NAME: $WEBUI_NAME })}
									{:else if mode === 'signin'}
										{$i18n.t(`Sign in to {{WEBUI_NAME}}`, { WEBUI_NAME: $WEBUI_NAME })}
									{:else}
										{$i18n.t(`Sign up to {{WEBUI_NAME}}`, { WEBUI_NAME: $WEBUI_NAME })}
									{/if}
								</div>
							</div>

							{#if mode === 'signup'}
								<div class="mb-4">
									<input
										type="text"
										class="w-full px-3 py-2 bg-black/30 dark:bg-black/30 border border-emerald-900/30 dark:border-emerald-700/30 rounded-xl focus:outline-none focus:border-emerald-500/50 dark:focus:border-emerald-500/50 placeholder-gray-500 text-gray-200"
										placeholder={$i18n.t('Enter your name')}
										bind:value={name}
									/>
								</div>
							{/if}

							{#if mode === 'ldap'}
								<div class="mb-4">
									<input
										type="text"
										class="w-full px-3 py-2 bg-black/30 dark:bg-black/30 border border-emerald-900/30 dark:border-emerald-700/30 rounded-xl focus:outline-none focus:border-emerald-500/50 dark:focus:border-emerald-500/50 placeholder-gray-500 text-gray-200"
										placeholder={$i18n.t('Enter your LDAP username')}
										bind:value={ldapUsername}
									/>
								</div>
							{:else}
								<div class="mb-4">
									<input
										type="email"
										class="w-full px-3 py-2 bg-black/30 dark:bg-black/30 border border-emerald-900/30 dark:border-emerald-700/30 rounded-xl focus:outline-none focus:border-emerald-500/50 dark:focus:border-emerald-500/50 placeholder-gray-500 text-gray-200"
										placeholder={$i18n.t('Enter your email')}
										bind:value={email}
									/>
								</div>
							{/if}

							<div class="mb-4">
								<input
									type="password"
									class="w-full px-3 py-2 bg-black/30 dark:bg-black/30 border border-emerald-900/30 dark:border-emerald-700/30 rounded-xl focus:outline-none focus:border-emerald-500/50 dark:focus:border-emerald-500/50 placeholder-gray-500 text-gray-200"
									placeholder={$i18n.t('Enter your password')}
									bind:value={password}
								/>
							</div>

							<button
								type="submit"
								class="w-full py-2 px-4 bg-gradient-to-r from-emerald-600 to-purple-600 hover:from-emerald-500 hover:to-purple-500 text-white font-medium rounded-xl transition-colors duration-300 shadow-lg hover:shadow-emerald-500/20"
							>
								{#if mode === 'ldap'}
									{$i18n.t('Sign in with LDAP')}
								{:else if mode === 'signin'}
									{$i18n.t('Sign in')}
								{:else}
									{$i18n.t('Sign up')}
								{/if}
							</button>

							{#if Object.keys($config?.oauth?.providers ?? {}).length > 0}
								<div class="inline-flex items-center justify-center w-full mt-6">
									<hr class="w-32 h-px border-0 dark:bg-emerald-700/20 bg-emerald-300/20" />
									<span class="absolute px-3 text-sm font-medium text-gray-400 bg-black dark:bg-black">
										{$i18n.t('or')}
									</span>
								</div>

								<div class="flex flex-col space-y-3 mt-6">
									{#if $config?.oauth?.providers?.google}
										<button
											class="flex justify-center items-center bg-black/30 hover:bg-black/40 dark:bg-black/30 dark:hover:bg-black/40 dark:text-gray-300 dark:hover:text-white transition w-full rounded-xl font-medium text-sm py-2.5 border border-emerald-900/30 dark:border-emerald-700/30"
											on:click={() => {
												window.location.href = `${WEBUI_BASE_URL}/oauth/google/login`;
											}}
										>
											<img
												crossorigin="anonymous"
												src="{WEBUI_BASE_URL}/static/google.svg"
												class="w-5 h-5 mr-2"
												alt="google"
											/>
											{$i18n.t('Continue with Google')}
										</button>
									{/if}

									{#if $config?.oauth?.providers?.github}
										<button
											class="flex justify-center items-center bg-black/30 hover:bg-black/40 dark:bg-black/30 dark:hover:bg-black/40 dark:text-gray-300 dark:hover:text-white transition w-full rounded-xl font-medium text-sm py-2.5 border border-emerald-900/30 dark:border-emerald-700/30"
											on:click={() => {
												window.location.href = `${WEBUI_BASE_URL}/oauth/github/login`;
											}}
										>
											<img
												crossorigin="anonymous"
												src="{WEBUI_BASE_URL}/static/github.svg"
												class="w-5 h-5 mr-2 dark:invert"
												alt="github"
											/>
											{$i18n.t('Continue with GitHub')}
										</button>
									{/if}

									{#if $config?.oauth?.providers?.azure}
										<button
											class="flex justify-center items-center bg-black/30 hover:bg-black/40 dark:bg-black/30 dark:hover:bg-black/40 dark:text-gray-300 dark:hover:text-white transition w-full rounded-xl font-medium text-sm py-2.5 border border-emerald-900/30 dark:border-emerald-700/30"
											on:click={() => {
												window.location.href = `${WEBUI_BASE_URL}/oauth/azure/login`;
											}}
										>
											<img
												crossorigin="anonymous"
												src="{WEBUI_BASE_URL}/static/microsoft.svg"
												class="w-5 h-5 mr-2"
												alt="microsoft"
											/>
											{$i18n.t('Continue with Microsoft')}
										</button>
									{/if}

									{#if $config?.oauth?.providers?.oidc}
										<button
											class="flex justify-center items-center bg-black/30 hover:bg-black/40 dark:bg-black/30 dark:hover:bg-black/40 dark:text-gray-300 dark:hover:text-white transition w-full rounded-xl font-medium text-sm py-2.5 border border-emerald-900/30 dark:border-emerald-700/30"
											on:click={() => {
												window.location.href = `${WEBUI_BASE_URL}/oauth/oidc/login`;
											}}
										>
											<img
												crossorigin="anonymous"
												src="{WEBUI_BASE_URL}/static/openid.svg"
												class="w-5 h-5 mr-2"
												alt="openid"
											/>
											{$i18n.t('Continue with OpenID Connect')}
										</button>
									{/if}
								</div>
							{/if}

							{#if $config?.features.enable_ldap && $config?.features.enable_login_form}
								<div class="mt-4">
									<button
										type="button"
										class="text-sm text-gray-400 hover:text-gray-300"
										on:click={() => {
											mode = mode === 'ldap' ? 'signin' : 'ldap';
										}}
									>
										{#if mode === 'ldap'}
											{$i18n.t('Sign in with Email')}
										{:else}
											{$i18n.t('Sign in with LDAP')}
										{/if}
									</button>
								</div>
							{/if}

							{#if mode === 'signin' && $config?.features.enable_signup}
								<div class="mt-4">
									<button
										type="button"
										class="text-sm text-gray-400 hover:text-gray-300"
										on:click={() => {
											mode = 'signup';
										}}
									>
										{$i18n.t('Create an account')}
									</button>
								</div>
							{:else if mode === 'signup'}
								<div class="mt-4">
									<button
										type="button"
										class="text-sm text-gray-400 hover:text-gray-300"
										on:click={() => {
											mode = 'signin';
										}}
									>
										{$i18n.t('Already have an account?')}
									</button>
								</div>
							{/if}
						</form>
					</div>
				{/if}
			</div>
		</div>
	{/if}
</div>

<style>
	.animate-gradient {
		background-size: 200% auto;
		animation: gradient 8s linear infinite;
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
</style>
