<script context="module">
  export function preload() {
    return this.fetch('https://svelte-course-1a51f-default-rtdb.firebaseio.com/meetups.json')
      .then(res => {
        if (!res.ok) {
          throw new Error('An error occurred, please try again!')
        }
        return res.json()
      })
      .then(data => {
        const loadedData = []
        for (const key in data) {
          loadedData.push({
            ...data[key],
            id: key
          })
        }
        return { fetchedMeetups: loadedData.reverse() }
      })
      .catch(err => {
        this.error(500, 'Could not fetch meetups!')
      })
  }
</script>

<script>
  import { onMount, onDestroy } from 'svelte'
  import { scale } from 'svelte/transition'
  import { flip } from 'svelte/animate'
  import MeetupsStore from '../store/meetups-store.js'
  import MeetupItem from '../components/Meetup/MeetupItem.svelte'
  import MeetupFilter from '../components/Meetup/MeetupFilter.svelte'
  import Button from '../components/UI/Button.svelte';
  import EditMeetup from '../components/Meetup/EditMeetup.svelte'
  import Loading from '../components/UI/Loading.svelte'

  export let fetchedMeetups
  
  let editMode = null
  let editedId = null
  let isLoading = null
  let favsOnly = false
  let unsubscribe

  $: filteredMeetups = favsOnly ? fetchedMeetups.filter(m => m.isFavorite) : fetchedMeetups

  onMount(() => {
    MeetupsStore.setMeetups(fetchedMeetups)
    unsubscribe = MeetupsStore.subscribe(items => {
      fetchedMeetups = items
    })
  })

  onDestroy(() => {
    if (unsubscribe) {
      unsubscribe()
    }
  })

  let saveMeetup = () => {
    editMode = null
    editedId = null
  }

  let cancelMeetup = () => {
    editMode = null
    editedId = null
  }

  let startAdd = () => {
    editMode = 'edit'
  }

  let startEdit = (event) => {
    editMode = 'edit'
    editedId = event.detail
  }

  let setFilter = (event) => {
    favsOnly = event.detail === 1
  }
</script>

<svelte:head>
  <title>All Meetups</title>
</svelte:head>

{#if editMode === 'edit'}
  <EditMeetup id={editedId} on:save-meetup={saveMeetup} on:cancel-modal={cancelMeetup} />
{/if}

{#if isLoading}
  <Loading />
{:else}
  <section id="filters">
    <MeetupFilter on:filter={setFilter} />
    <Button on:click={startAdd}>New Meetup</Button>
  </section>

  {#if filteredMeetups.length === 0}
    <p class="no-meetups">No meetups found, you can start adding some.</p>
  {:else}
    <section id="meetups">
      {#each filteredMeetups as meetup (meetup.id)}
        <div transition:scale animate:flip={{duration: 300}}>
          <MeetupItem
            id={meetup.id}
            title={meetup.title}
            subtitle={meetup.subtitle}
            description={meetup.description}
            imageUrl={meetup.imageUrl}
            address={meetup.address}
            isFav={meetup.isFavorite}
            on:edit-meetup={startEdit}
          />
        </div>
      {/each}
    </section>
  {/if}
{/if}

<style>
  #meetups {
    width: 100%;
    display: grid;
    grid-template-columns: 1fr;
    grid-gap: 1rem;
  }

  #filters {
    margin: 1rem;
    display: flex;
    justify-content: space-between;
  }

  .no-meetups {
    display: flex;
    justify-content: center;
    align-items: center;
  }

  @media (min-width: 768px) {
    #meetups {
      grid-template-columns: repeat(2, 1fr);
    }
  }
</style>