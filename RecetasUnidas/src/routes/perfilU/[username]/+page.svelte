<script lang="ts">
	import { page } from '$app/stores';
	import { supabase } from '$lib/supabaseClient';
	import { afterUpdate, beforeUpdate, onMount } from 'svelte';
	import Nav from '../../../components/Nav.svelte';
	import food_stand_day from '/src/lib/assets/food-stand-day.png';

	let desc = '';
	let dataRecetas: any[] = [];
	let profilePicture: any;

	const { params } = $page;
	const username = params.username;
	onMount(async () => {
		const logoElement = (document.getElementById('logo') as HTMLElement) || null;

		logoElement.addEventListener('click', () => {
			window.location.href = '/feed';
		});

		// Obtén el usuario
		const user = await supabase.from('usuarios').select('*').eq('nombreusuario', username);
		desc = user.data?.[0]?.descripcion;
		// Si el usuario está autenticado, obtén su ID
		// Query para obtener las recetas del usuario
		const { data } = await supabase
			.from('recetas')
			.select('*')
			.eq('idusuario', user.data?.[0]?.usuario_uuid);

		if (data) {
			dataRecetas = data;
		}
	});

	afterUpdate(() => {
		for (let receta of dataRecetas) {
			handleImageRecovery(receta.idreceta, receta.tituloreceta);
		}
	});

	beforeUpdate(() => {
		profilePicture = "https://kaonlhtranrfojpknofp.supabase.co/storage/v1/object/public/fotosPerfil/default.png";
		fetchProfilePicture();
	});

	const fetchProfilePicture = async () => {
		// First, get the user's ID from the user recovered from the database
		const user = await supabase.from('usuarios').select('*').eq('nombreusuario', username);
		const userID = user.data?.[0]?.usuario_uuid

		// Then, get the user's profile picture
		const { data: imagen } = await supabase.storage
			.from('fotosPerfil')
			.getPublicUrl(`${userID}.png`);

		if (imagen) {
			profilePicture = imagen.publicUrl;
		} else {
			profilePicture = await supabase.storage.from('fotosPerfil').getPublicUrl('default.png').data
				.publicUrl;
		}
	};

	supabase.auth.onAuthStateChange((event) => {
		if (event === 'SIGNED_OUT') {
			window.location.href = '/login';
		}
	});

	import Modal from './modal.svelte';

	let isOpen = false;

	function openModal() {
		isOpen = true;
	}

	function closeModal() {
		isOpen = false;
	}

	const handleImageRecovery = async (idReceta: number, nombreReceta: string) => {
		// console.log("Receta ID: ", idReceta, "Receta Name: ", nombreReceta);
		let imagenReceta =
			(document.getElementById(`imagenReceta-${idReceta}`) as HTMLImageElement) || null;
		// console.log(imagenReceta.alt);
		// const { data } = supabase.storage.from('public-bucket').getPublicUrl('/avatar1.png');
		// Following the example above, you can get the public URL of the image and set it to the imageRecetaURL variable, the name of the image is the same as the name of the recipe but in lowercase and spaced with hyphens

		//const nombreImagen = data.recetas[0].tituloreceta.toLowerCase().replace(/ /g, '-');
		// For each recipe, get the image

		let nombreImagen: any;
		nombreImagen = nombreReceta.toLowerCase().replace(/ /g, '-');
		// console.log(nombreImagen);

		// The file format can be anything, like .png, .jpg, .jpeg, etc.
		const { data: imagen } = await supabase.storage
			.from('fotosRecetas')
			.getPublicUrl(`${nombreImagen}.png`);

		// console.log(imagen.publicUrl);

		for (let receta of dataRecetas) {
			if (receta.idreceta === idReceta) {
				if (imagen) {
					imagenReceta.src = imagen.publicUrl;
				} else {
					imagenReceta.src = food_stand_day;
				}
			}
		}
	};
</script>

<Nav />
<body>
	<div class="container">
		<section id="profileSection">
			<h1>{username}</h1>
			<Modal bind:isOpen src={profilePicture} on:close={closeModal} />
			<button id="profilePictureButton" on:click={openModal}>
				<img id="profilePicture" src={profilePicture} alt="Foto de perfil" />
			</button>
			{#if desc}
				<p id="descripcion">{desc}</p>
			{/if}
		</section>
		<section id="publicacion_section">
			<h2>Publicaciones</h2>
			<div id="publicaciones_container">
				{#each dataRecetas as receta}
					<div id="publicacion">
						<div>
							<h3>{receta.tituloreceta}</h3>
							<span>{receta.descripcionreceta}</span>
							<p><span>Tiempo de preparación:</span> {receta.tiempopreparacionreceta} minutos</p>
							<p><span>Dificultad:</span> {receta.dificultadreceta}</p>
						</div>
						<div>
							<img alt={receta.tituloreceta} id={`imagenReceta-${receta.idreceta}`} />
						</div>
						<div id="recipeButtonContainer">
							<button
								id="recipeButton"
								on:click={() => (window.location.href = `/receta/${receta.idreceta}`)}
								>Ver Receta
							</button>
						</div>
					</div>
				{/each}
			</div>
		</section>
	</div>
</body>

<style>
	.container {
		display: grid;
		grid-template-columns: 30% 70%;
		height: 100%;
	}

	h1 {
		font-size: 3rem;
	}

	#descripcion {
		width: 80%;
		font-size: 1.75rem;
		text-align: center;
	}
	#publicacion {
		border: 1px solid #363636;
		padding: 0.5rem 1rem;
		width: 100%;
		display: grid;
		grid-template-columns: repeat(2, 1fr);
		gap: 1rem;
	}

	#publicacion p {
		text-transform: capitalize;
	}

	#publicacion p > span {
		font-weight: 600;
	}

	#publicacion div:nth-child(2) img {
		border-radius: 0.5rem;
		width: 20rem;
		aspect-ratio: 7/5;
	}

	#publicaciones_container {
		width: 100%;
		display: grid;
		grid-template-columns: repeat(1, 1fr);
		gap: 2rem;
		padding: 0 1rem;
	}

	#publicacion > div:nth-child(1) {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: flex-start;
		gap: 1rem;
	}

	#publicacion > div:nth-child(2) {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: flex-end;
	}

	#recipeButtonContainer {
		grid-column: 1/-1;
		display: flex;
		justify-content: flex-start;
		align-items: center;
	}

	#publicacion #recipeButton {
		border: none;
		width: 8rem;
		height: fit-content;
		padding: 0.5rem;
		border-radius: 2rem;
		background: #8b0000;
		color: #fff;
		text-align: center;
		font-weight: 700;
		transition: background-color 0.2s ease-in-out;
	}

	#publicacion #reciteButton:hover {
		cursor: pointer;
		background: #A52A2A;
	}

	#publicacion_section {
		width: 100%;
		padding: 1rem;
		display: flex;
		flex-direction: column;
		justify-content: flex-start;
		align-items: flex-start;
		gap: 1rem;
	}

	#profileSection {
		padding: 2rem;
		background-color: #698497;
		display: flex;
		flex-direction: column;
		justify-content: flex-start;
		align-items: center;
		gap: 2rem;
	}

	#profileSection p {
		font-size: 1.575rem;
		font-weight: 450;
	}

	#profilePictureButton {
		background-color: transparent;
		border: none;
		border-radius: 10rem;
		display: flex;
		flex-direction: row;
		justify-content: center;
		align-items: center;
	}

	#profilePictureButton:hover {
		cursor: pointer;
	}

	#profilePicture {
		border-radius: 100%;
		width: 20rem;
		height: 20rem;
		border: #000 2px solid;
	}
</style>
