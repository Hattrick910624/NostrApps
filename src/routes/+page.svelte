<script lang="ts">
	import { browser } from '$app/environment';
	import { each } from 'svelte/internal';
	import NDK, { NDKEvent, NDKNip07Signer } from '@nostr-dev-kit/ndk';
	import type { NDKUserProfile } from '@nostr-dev-kit/ndk';
	import { nip19 } from 'nostr-tools';

	const ndk = new NDK({
		explicitRelayUrls: [
			'wss://relay.snort.social',
			'wss://relay.nostr.band',
			'wss://relay.damus.io'
		]
	});

	let userProfile: NDKUserProfile;
	let event: NDKEvent | null;

	if (browser) {
		ndk.connect().then(() => {
			console.log('Connected');
			fetchEventFromId();
			fetchEventFromSub();
		});
	}
	
	function fetchEventFromSub() {
		const sub = ndk.subscribe({kinds:[1]}, { closeOnEose :false });

		sub.on('event', (event) => {
			console.log(event);
		});

		sub.on('eose', () => {
			console.log('EOSE');
		});

		sub.on('notice', (notice) => {
			console.log(notice);
		});
	}

	function fetchEventFromId() {
		const noteId = "note1fe0d684zx6m928dfsa8ejzp3ydnez5zjc9qu0y7r24jcnc5uq8nq0kdash";
		const decoded = nip19.decode(noteId);
		ndk.fetchEvent({ids:[decoded.data], kinds: [1]}).then((fetchedEvent) => {
			console.log(event);
			event = fetchedEvent;
		});
	}

	async function login() {
	// 	const user = ndk.getUser({
	// 	npub: 'npub1vmwell0r2708nk6uta8ld0jl0lp35jysdwhm5kj4x640uz7md4gsm32whk'
	// });
		const signer = new NDKNip07Signer();
		ndk.signer = signer;
		signer.user().then((user) => {
			user.ndk = ndk;
			user.fetchProfile().then((eventSet) => {
               console.log(user);
			   userProfile = user.profile as NDKUserProfile;
			});
		});
	}

	

	// const eventPromise = ndk.fetchEvents({ kinds: [1], authors: [user.hexpubkey] });
</script>


<button on:click={login} 
        class="px-3 py-2 border-1 border-purple-900 bg-purple-800 hover:bg-purple-700 rounded-md">
		Log in with Nostr
</button>

{#if userProfile}
    <div 
		class="flex flex-row gap-4 items-center my-8 p-6 rounded-md bg-slate-800 border-slate">
	    <img src={userProfile.image} alt="profilepic" class="rounded-full w-20 h-20 m-0"/>
		<div>
			<h2 class="mt-0 mb-2 flex flex-row items-center gap-2">
				{userProfile.name}
			</h2>
			<span>{userProfile.about}</span>
		</div>
	</div>
{/if}

<!-- {#await user.fetchProfile() then events}
	<h2>{user.profile?.name}</h2>
	<p>
		<img src={user.profile?.image} style="width:100px; height:100px;" alt="asdf" />
	</p>
	<p>{user.profile?.about}</p>
{/await}

{#await eventPromise then events}
	{#each Array.from(events) as event}
		<div class="eventBlock">
			<p>{event.content}</p>
		</div>
	{/each}
{/await}

<style>
	.eventBlock {
		padding: 10px;
		border: 1px black;
		border-radius: 10px;
		margin-bottom: 4px;
	}
</style> -->
