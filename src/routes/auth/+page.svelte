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
	<link href="https://fonts.googleapis.com/css2?family=ZCOOL+XiaoWei&family=Noto+Serif+SC:wght@400;500&display=swap" rel="stylesheet">
</svelte:head>

<style>
	.animate-gradient {
		background-size: 200% auto;
		animation: gradient 8s linear infinite;
	}

	.animate-border {
		position: relative;
		backdrop-filter: blur(12px);
		background: rgba(0, 0, 0, 0.4);
		border: 1px solid rgba(255, 255, 255, 0.1);
		box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.37);
		transition: all 0.3s ease;
	}

	.animate-border:hover {
		background: rgba(0, 0, 0, 0.5);
		border: 1px solid rgba(255, 255, 255, 0.15);
		box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.45);
	}

	.animate-border::before {
		content: '';
		position: absolute;
		inset: -1px;
		background: linear-gradient(
			45deg,
			rgba(141, 255, 249, 0.1),
			rgba(141, 255, 249, 0.2),
			rgba(141, 255, 249, 0.3),
			rgba(141, 255, 249, 0.2),
			rgba(141, 255, 249, 0.1)
		);
		background-size: 200% 200%;
		animation: borderGradient 4s linear infinite;
		border-radius: 1rem;
		z-index: -1;
	}

	@keyframes borderGradient {
		0% { background-position: 0% 0%; }
		50% { background-position: 100% 100%; }
		100% { background-position: 0% 0%; }
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

	.input-mystic {
		background: rgba(255, 255, 255, 0.05);
		border: 1px solid rgba(255, 255, 255, 0.1);
		backdrop-filter: blur(5px);
		transition: all 0.3s ease;
	}

	.input-mystic:hover, .input-mystic:focus {
		background: rgba(255, 255, 255, 0.08);
		border: 1px solid rgba(255, 255, 255, 0.2);
	}

	.btn-mystic {
		background: linear-gradient(45deg, rgba(78, 205, 196, 0.1), rgba(69, 183, 209, 0.1));
		border: 1px solid rgba(255, 255, 255, 0.1);
		backdrop-filter: blur(5px);
		transition: all 0.3s ease;
	}

	.btn-mystic:hover {
		background: linear-gradient(45deg, rgba(78, 205, 196, 0.2), rgba(69, 183, 209, 0.2));
		border: 1px solid rgba(255, 255, 255, 0.2);
		transform: translateY(-1px);
		box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
	}

	.mode-switch {
		position: relative;
		overflow: hidden;
	}

	.mode-switch-enter {
		animation: slideIn 0.3s ease forwards;
	}

	.mode-switch-exit {
		animation: slideOut 0.3s ease forwards;
	}

	@keyframes slideIn {
		from {
			opacity: 0;
			transform: translateY(10px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	@keyframes slideOut {
		from {
			opacity: 1;
			transform: translateY(0);
		}
		to {
			opacity: 0;
			transform: translateY(-10px);
		}
	}
</style>

<OnBoarding
	bind:show={onboarding}
	getStartedHandler={() => {
		onboarding = false;
		mode = $config?.features.enable_ldap ? 'ldap' : 'signup';
	}}
/>

<div class="w-full h-screen max-h-[100dvh] text-white relative">
	<div class="w-full h-full absolute top-0 left-0 bg-black/90"></div>

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
					<div class="absolute inset-0 rounded-xl shadow-[0_0_15px_rgba(0,255,255,0.3)] group-hover:shadow-[0_0_25px_rgba(0,255,255,0.4)] transition-shadow duration-300"></div>
				</div>
				<div class="text-2xl font-medium mystic-text">
					{$WEBUI_NAME}
				</div>
			</div>
		</div>

		<div class="fixed bg-transparent min-h-screen w-full flex justify-center font-primary z-40">
			<div 
				class="w-full sm:max-w-md px-10 min-h-screen flex flex-col text-center"
				in:fly={{ y: 20, duration: 600, delay: 200 }}
			>
				{#if ($config?.features.auth_trusted_header ?? false) || $config?.features.auth === false}
					<div class="my-auto pb-10 w-full">
						<div class="flex items-center justify-center gap-3 text-xl sm:text-2xl text-center font-semibold text-cyan-200 mystic-text">
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
							class="flex flex-col justify-center p-10 rounded-2xl animate-border space-y-6"
							on:submit={(e) => {
								e.preventDefault();
								submitHandler();
							}}
						>
							<div class="mb-2">
								<div class="text-3xl font-medium mystic-text mode-switch">
									{#if mode === 'ldap'}
										{$i18n.t(`Sign in to {{WEBUI_NAME}} with LDAP`, { WEBUI_NAME: $WEBUI_NAME })}
									{:else if mode === 'signup'}
										{$i18n.t(`Sign up to {{WEBUI_NAME}}`, { WEBUI_NAME: $WEBUI_NAME })}
									{:else}
										{$i18n.t(`Sign in to {{WEBUI_NAME}}`, { WEBUI_NAME: $WEBUI_NAME })}
									{/if}
								</div>
							</div>

							{#if mode === 'ldap'}
								<div class="mb-4">
									<input
										type="text"
										class="w-full p-2 rounded-lg input-mystic"
										placeholder={$i18n.t('LDAP Username')}
										bind:value={ldapUsername}
									/>
								</div>
							{:else}
								{#if mode === 'signup'}
									<div class="mb-4">
										<input
											type="text"
											class="w-full p-2 rounded-lg input-mystic"
											placeholder={$i18n.t('Name')}
											bind:value={name}
										/>
									</div>
								{/if}
								<div class="mb-4">
									<input
										type="email"
										class="w-full p-2 rounded-lg input-mystic"
										placeholder={$i18n.t('Email')}
										bind:value={email}
									/>
								</div>
							{/if}

							<div class="mb-4">
								<input
									type="password"
									class="w-full p-2 rounded-lg input-mystic"
									placeholder={$i18n.t('Password')}
									bind:value={password}
								/>
							</div>

							<button type="submit" class="w-full p-2 rounded-lg btn-mystic">
								{#if mode === 'ldap'}
									{$i18n.t('Sign in with LDAP')}
								{:else if mode === 'signup'}
									{$i18n.t('Sign up')}
								{:else}
									{$i18n.t('Sign in')}
								{/if}
							</button>

							{#if mode !== 'ldap' && $config?.features.auth === true}
								<div class="mt-4">
									<button
										type="button"
										class="text-sm text-cyan-200/90 hover:text-cyan-100 transition-colors duration-300 mystic-text"
										on:click={() => {
											const oldMode = mode;
											mode = mode === 'signin' ? 'signup' : 'signin';
											const title = document.querySelector('.mode-switch');
											if (title) {
												title.classList.add('mode-switch-exit');
												setTimeout(() => {
													title.classList.remove('mode-switch-exit');
													title.classList.add('mode-switch-enter');
												}, 300);
											}
										}}
									>
										{#if mode === 'signin'}
											{$i18n.t("Don't have an account? Sign up")}
										{:else}
											{$i18n.t('Already have an account? Sign in')}
										{/if}
									</button>
								</div>
							{/if}

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
