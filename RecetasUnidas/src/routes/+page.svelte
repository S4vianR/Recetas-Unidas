<script lang="ts">
	// Image imports
	import { supabase } from '$lib/supabaseClient';
	import { onMount } from 'svelte';
	import logo from '/src/lib/assets/logo-removebg.png';

	// Variables
	let correo = '';
	let password = '';

	onMount(() => {
		correo = (document.getElementById('correo') as HTMLInputElement)?.value || '';
		password = (document.getElementById('password') as HTMLInputElement)?.value || '';
	});

	// Sign in
	async function signInWithEmail() {
		const { data, error } = await supabase.auth.signInWithPassword({
			email: correo,
			password: password
		});
		if (error) {
			alert('Error al iniciar sesión');
		} else {
			window.location.href = '/feed';
		}
	}
</script>

<main class="white">
	<section>
		<section class="left_pane">
			<img id="logo" src={logo} class="logo" alt="Logo" />
		</section>
		<section class="right_pane">
			<h2>Inicio de sesión</h2>
			<form on:submit={signInWithEmail} method="get">
				<section>
					<div>
						<label for="username">Correo</label>
						<input type="text" name="username" id="username" required bind:value={correo} />
					</div>
					<div>
						<label for="password">Contraseña</label>
						<input type="password" name="password" id="password" required bind:value={password} />
					</div>
					<button type="submit"> Iniciar sesión</button>
					<div id="mensaje_registro">
						<p>¿No tienes una cuenta?</p>
						<a href="/registro">¡Crea una ahora mismo!</a>
					</div>
				</section>
			</form>
		</section>
	</section>
</main>

<style>
	main > section:nth-child(1) {
		display: grid;
		grid-template-columns: 40rem auto;
		height: 100svh;
	}

	.left_pane {
		display: flex;
		justify-content: center;
		align-items: center;
		padding: 1rem;
		background-color: #36454F;
	}


	.right_pane {
		padding: 1rem;
		display: flex;
		flex-direction: column;
		gap: 5rem;
		justify-content: flex-start;
		align-items: center;
		margin-top: 5rem;
	}

	.right_pane > h2 {
		text-align: center;
		font-size: 2rem;
		font-weight: 700;
	}

	.right_pane > form {
		border: 3px solid #000;
		padding: 2rem;
		width: 55%;
		aspect-ratio: 3/2;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}

	.right_pane > form > section {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: normal;
		gap: 4rem;
		width: fit-content;
	}

	.right_pane > form div {
		display: flex;
		flex-direction: column;
		justify-content: center;
		gap: 0.5rem;
		align-items: center;
	}

	.right_pane > form label {
		font-size: 1.5rem;
		font-weight: 600;
	}

	.right_pane > form input[type='text'],
	.right_pane > form input[type='password'] {
		background-color: #cecece;
		outline: none;
		border: 0.12rem solid #3f3f3f;
		border-radius: 0.5rem;
		width: 100%;
		height: 2.575rem;
		flex-shrink: 0;
		padding: 0.5rem;
		font-size: 1rem;
	}

	.right_pane > form button[type='submit'] {
		border: 1px solid #000;
		border-radius: 0.5rem;
		background-color: #c99669;
		height: 2rem;
		width: fit-content;
		font-size: 1rem;
		font-weight: 500;
		padding: 0 1rem 0 1rem;
		align-self: flex-end;
	}

	.right_pane > form button[type='submit']:hover {
		background-color: #c57b31;
		cursor: pointer;
	}

	.right_pane > form #mensaje_registro {
		display: flex;
		flex-direction: row;
		justify-content: center;
		align-items: center;
		gap: 0.3rem;
		font-weight: 600;
	}

	#mensaje_registro > a {
		color: #801c02;
	}
	.white {
		background-color: #faf9f6;
	}
</style>
