<script>
  import { onMount } from "svelte";
  const url = "http://104.199.88.2/api/translate";
  let languages = [];

  const getLanguages = async () => {
    try {
      const res = await fetch(url);
      const result = await res.json();
      languages = result.languages;
    } catch (err) {
      console.dir(err);
    }
  };

  let languageTo = "";
  let languageFrom = "";
  let text = "";
  let translation = "";
  let err = "";

  onMount(() => {
    getLanguages();
    const savedText = localStorage.getItem("text");
    if (savedText) text = savedText;
  });

  $: if (text || text === "") localStorage.setItem("text", text);

  const handleSubmit = async e => {
    e.preventDefault();
    if (languageFrom === languageTo) {
      err = "The language to and from need to be different";
      return;
    }
    try {
      const response = await fetch(url, {
        method: "POST",
        body: JSON.stringify({
          text,
          from: languageFrom,
          to: languageTo
        }),
        headers: {
          "Content-Type": "application/json"
        }
      });
      const res = await response.json();
      if (res.statusCode > 400) throw res;
      else {
        console.log("object");
        const { translation: responseTranslation } = res;
        translation = responseTranslation;
      }
    } catch (error) {
      const { message } = error;
      console.log(error);
      err = message;
    }
  };
</script>

<style>
  .header {
    background: #202829;
    top: 0;
    height: 7vh;
    width: 100vw;
    display: flex;
  }

  .header > h1 {
    vertical-align: middle;
    margin: auto;
    color: white;
  }

  .title {
    margin-block-end: 0;
    margin-block-start: 0;
    margin-inline-end: 0;
    margin-inline-start: 0;
    color: blanchedalmond;
  }

  main {
    /* background: inherit; */
    height: 93vh;
    overflow: scroll;
  }

  .main-form {
    /* background: inherit; */

    display: grid;
    height: 100%;
    overflow: scroll;
    grid-template-rows: 2fr 8fr 1fr;
    justify-items: center;
  }

  .language-input {
    width: 80%;
    display: flex;
    justify-content: space-evenly;
    align-items: center;
    flex-wrap: wrap;
    min-width: 100%;
  }

  .language-input > label {
    background: white;
    width: 40%;
    height: 60%;
    display: grid;
  }

  .left,
  .right {
    width: 45%;
    background: white;
    display: flex;
    align-items: center;
    justify-content: space-around;
    flex-direction: row;
    height: 35%;
    min-width: 290px;
    margin: 2%;
  }

  .left {
    /* margin-top: 4%; */
  }

  #from,
  #to {
    padding: 0;
    margin: 0;
    width: 60%;
  }

  .translation-text {
    display: flex;
    justify-content: space-evenly;
    width: 100%;
    flex-wrap: wrap;
  }

  .translation-input,
  .translation-output {
    width: 45%;
    background: white;
    height: 45%;
    min-width: 290px;
    margin: 2%;
  }

  .translation-input {
    /* margin-top: 4%; */
  }

  .translation-input > label,
  .translation-output > label {
    display: grid;
    align-content: center;
    text-align: start;
    padding-left: 5%;
    height: 12%;
  }
  textarea {
    width: 90%;
    height: 80%;
  }
</style>

<header class="header">
  <h1 class="title">Shaqs translations</h1>
</header>
<main>
  {#if err}
    <h3>{err}</h3>
  {/if}
  <form on:submit={handleSubmit} class="main-form">
    <div class="language-input">
      <div class="left">
        <label for="from">Translate from:</label>
        <select name="from" id="from" bind:value={languageFrom} required>
          {#each languages as { language, name }}
            <option value={language}>{name}</option>
          {/each}
        </select>
      </div>
      <div class="right">
        <label for="to">Translate to:</label>
        <select name="to" id="to" bind:value={languageTo} required>
          {#each languages as { language, name }}
            <option value={language}>{name}</option>
          {/each}
        </select>
      </div>
    </div>
    <div class="translation-text">
      <div class="translation-input">
        <label for="input">Text:</label>
        <textarea type="text" required bind:value={text} id="input" />
      </div>
      <div class="translation-output">
        <label for="output">Translation:</label>
        <textarea type="text" bind:value={translation} id="output" />
      </div>
    </div>
    <button>Translate</button>
  </form>
</main>
