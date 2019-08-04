<script>
  let likedPlaces = [];
  let dislikedPlaces = [];

  let places = [];
  let filteredPlaces = [];

  const promise = getPlaces();

  async function getPlaces() {
    const response = await fetch("/utno-bergen.json");

    if (response.ok) {
      places = (await response.json()).places.documents;
      filteredPlaces = filterPlaces();

      return places;
    } else {
      throw new Error(await response.text());
    }
  }

  function filterPlaces() {
    return places.filter(
      place => !likedPlaces.includes(place) && !dislikedPlaces.includes(place)
    );
  }

  function renderDistance(distance) {
    return `${distance / 1000} kilometres away`;
  }

  function renderPoiType(poiTypes) {
    const poiTypeNames = poiTypes
      .map(type => capitalize(type.name))
      .filter(name => name.length > 0);

    return poiTypeNames.join(", ");
  }

  function capitalize(text) {
    return text.substring(0, 1).toUpperCase() + text.substring(1, text.length);
  }

  function renderImageSrcSets(versions) {
    const hasSmallerVersions = versions.length > 1;
    let srcsets = "";

    if (hasSmallerVersions) {
      srcsets = versions
        .map(version => `${version.url} ${version.width}w`)
        .join(",");
    }

    return srcsets;
  }

  function likePlace() {
    likedPlaces.push(filteredPlaces[0]);
    filteredPlaces = filterPlaces();

    nextPlace();
  }

  function dislikePlace() {
    dislikedPlaces.push(filteredPlaces[0]);
    filteredPlaces = filterPlaces();

    nextPlace();
  }

  function nextPlace() {
  }
</script>

<style>
  @import url("https://fonts.googleapis.com/css?family=Beth+Ellen&display=swap&text=mountider");
  .container {
    background-color: #f2f2f2;
    display: grid;
    grid-template-areas:
      "header"
      "content"
      "buttons";
    grid-template-rows: 100px 1fr 100px;
    height: 100vh;
  }

  .header {
    font-family: "Beth Ellen", cursive;
    font-size: 1.5rem;
    grid-area: header;
    padding: 2rem 0 1rem;
    text-align: center;
  }

  .content {
    grid-area: content;
    position: relative;
  }

  .place {
    background-color: #d0d0d0;
    background-image: url('/image-placeholder.png');
    background-position: center center;
    background-repeat: no-repeat;
    border-radius: 8px;
    box-shadow: 0 1px 5px rgba(0, 0, 0, .3);
    display: flex;
    flex-direction: column;
    height: 100%;
    overflow: hidden;
    position: absolute;
    transition: opacity 0.15s ease-in-out;
    width: 100%;
  }

  .place::after {
    background-image: linear-gradient(
      to top,
      rgba(0, 0, 0, 0.5),
      rgba(0, 0, 0, 0) 30%
    );
    content: "";
    height: 100%;
    left: 0;
    pointer-events: none;
    position: absolute;
    top: 0;
    width: 100%;
  }

  .place:not(:first-child) {
    opacity: 0;
    pointer-events: none;
  }

  .place-info {
    bottom: 0;
    color: #fff;
    padding: 1rem 1.5rem;
    position: absolute;
    z-index: 1;
  }

  .title {
    margin-top: 0;
  }

  .picture {
    flex-grow: 1;
    object-fit: cover;
  }

  .buttons {
    align-items: center;
    display: flex;
    grid-area: buttons;
    justify-content: center;
    text-align: center;
  }

  .action-button {
    border-radius: 50%;
    box-shadow: 0 1px 5px rgba(0, 0, 0, .3);
    cursor: pointer;
    font-size: 1.5rem;
    height: 4rem;
    width: 4rem;
  }

  .action-button:not(:first-child) {
    margin-left: 1rem;
  }
</style>

<div class="container">
  <header class="header">mountinder</header>
  <main class="content">
    {#await promise}
      <div class="loading">Loading</div>
    {:then _}
      {#if filteredPlaces.length > 0}
        <div class="places">
          {#each filteredPlaces as place}
            <article class="place">
              {#if place.pictures}
                {#each place.pictures as picture}
                  <img
                    class="picture"
                    srcset={renderImageSrcSets(picture.versions)}
                    src={picture.versions[0].url}
                    alt="" />
                {/each}
              {/if}
              <div class="place-info">
                <h2 class="title">{place.name}</h2>
                <div class="distance">
                  {renderDistance(place.near_distance_0)}
                </div>
                <div class="poi-type">{renderPoiType(place.poi_types)}</div>
              </div>
            </article>
          {/each}
        </div>
      {:else}
        <div class="places-empty-state">No more trails available</div>
      {/if}
    {:catch error}
      <div class="empty-state">No places found</div>
      <pre class="error">{error.message}</pre>
    {/await}
  </main>

  <div class="buttons">
    <button type="button" class="dislike action-button" on:click={dislikePlace}>
      💔
      <span class="visually-hidden">Dislike</span>
    </button>
    <button type="button" class="like action-button" on:click={likePlace}>
      💖
      <span class="visually-hidden">Like</span>
    </button>
  </div>
</div>
