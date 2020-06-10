<script>
	import Tailwindcss from './Tailwindcss.svelte';
	import { fade, fly } from 'svelte/transition';
	import { flip } from 'svelte/animate';
	import SearchBox from './SearchBox.svelte';
	let name = "";
	let stage = 0;
	let textHeight;
	let transitioning = false;
	let playerInfo = undefined;
	let copyText = "";
	let copyElem;
	let copied = false;

	function handleSubmit(event){
		name = event.detail.username;
		if (name.length > 0) {
			playerInfo = undefined;
			playerInfo = getUser();
		}
	}

	function isUUID(string){
		return string.match(/^[0-9a-f]{8}-[0-9a-f]{4}-[0-5][0-9a-f]{3}-[089ab][0-9a-f]{3}-[0-9a-f]{12}$/i) !== null;
	}

	async function getUser(){
		let apiEndpoint = "/username/";
		if(isUUID(name)){
			apiEndpoint = "/uuid/";
		} 
		const res = await fetch(`${process.env.isProd ? "https://api.mymcuu.id" : "http://localhost:8080"}${apiEndpoint}${name}`);
		const json = await res.json();
		if(res.ok) {
			if(json.error !== undefined) {
				throw new Error(json.error);
			} else {
				stage = 1;
				return json;
			}
		} else {
			throw new Error(json);
		}
	}

	function copyUUID(event){
		let target = event.target;
		copyText = target.children[0].innerHTML;
		copyElem.value = copyText;
		copyElem.hidden = false;
		copyElem.focus();
		copyElem.select();
		copyElem.setSelectionRange(0, 99999); 
		document.execCommand("copy");
		copyElem.hidden = true;
		copied = true;
		setTimeout(() => copied = false, 2000);
	}
	function copyUsername(event){
		let target = event.target;
		copyText = target.dataset.username;
		copyElem.value = copyText;
		copyElem.hidden = false;
		copyElem.focus();
		copyElem.select();
		copyElem.setSelectionRange(0, 99999); 
		document.execCommand("copy");
		copyElem.hidden = true;
		copied = true;
		setTimeout(() => copied = false, 2000);
	}
</script>
<main>
	{#if stage === 0}
	<div bind:clientHeight={textHeight} out:fade="{{duration: 1000}}" on:outrostart="{() => transitioning = true}" on:outroend="{() => transitioning = false}">
		<h1 class="font-bold text-gray-600 text-4xl md:text-6xl"  style="margin-top: 26vh;" >MyMCUU.ID</h1>
		<h2 out:fade>Type a username or UUID into the box to get it's profile</h2>
	</div>
	{/if}
	<div class="mt-10 {transitioning ? 'transition' : ''}" style="transform: translateY({transitioning ? -textHeight  : '0'}px);">
		<SearchBox username={name} on:submit={handleSubmit}></SearchBox>
	</div>
	
	{#if playerInfo !== undefined }
		{#await playerInfo}
			<p>Fetching information...</p>
		{:then player}
			<div class="mt-10 bg-white p-4 border rounded-lg w-3/4 md:w-1/2 mx-auto flex flex-col md:flex-row text-left" out:fade="{{duration: 500}}" in:fly="{{ y: 200, duration: 1000, delay: transitioning ? 1000 : 500}}">
				<div class="relative w-full md:w-1/5">
					<img src="{player.avatar}" alt="{player.username}'s avatar" class="h-full w-full"/>
				</div>
				<div class="px-2 flex flex-col flex-grow items-center md:items-stretch">
					<div  on:click={copyUsername} class="username flex flex-col cursor-pointer items-center md:flex-row" data-username="{player.username}">
						<h3 class="text-4xl  pointer-events-none">{player.username}</h3>
						<small class="uppercase text-sm select-none pointer-events-none"> Click to copy</small>
					</div>
					<div on:click={copyUUID} class="uuid-box bg-gray-300 rounded p-2 text-center cursor-pointer">
						<h4 class="text-xl select-none pointer-events-none">{player.uuid}</h4>
						<small class="uppercase text-sm select-none pointer-events-none">Click to copy</small>
					</div>
				</div>
			</div>
		{:catch error}
			<div class="mt-10 bg-red-300 p-4 border rounded-lg w-3/4 md:w-1/2 mx-auto flex flex-row text-left" out:fade="{{duration: 500}}" in:fly="{{ y: 200, duration: 1000, delay: transitioning ? 1000 : 500}}">
				<p>An error occurred {error}</p>
			</div>
		{/await}
	{/if}
</main>
{#if copied }
	<div class="absolute w-1/2 md:w-1/4 text-center bg-white mx-auto rounded-full p-4" style="bottom: 20px; left: 50%; transform: translate(-50%, -50%); box-shadow: 0px 3px 6px 0px rgba(0,0,0,0.16);" in:fly="{{y: 200}}" out:fade>
		Copied to Clipboard!
	</div>
{/if}
<textarea type="text" bind:this={copyElem} alt="You see nothing!" hidden>{copyText}</textarea>
<Tailwindcss />
<style>
	main {
		text-align: center;
		/* max-width: 240px; */
		width: 100%;
		display: flex;
		flex-direction: column;
		height: 100%;
		margin: 0 auto;
		font-family: acumin-pro, sans-serif;
		overflow-y: hidden;
	}
	h2 {
		font-size: 24px;
		color: #707070;	
	}
	h1 {
		color: #707070;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
	.transition {
		transition: transform 1s ease-in-out; 
	}
	.uuid-box > small {
		opacity: 0;
		transition: opacity .2s ease-in-out;
	} 
	.username > small {
		opacity: 0;
		transition: opacity .2s ease-in-out;
	}
	.username:hover > small {
		opacity: 1;
	}
	.uuid-box:hover > small {
		opacity: 1;
	}
</style>