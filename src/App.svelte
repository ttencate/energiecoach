<script lang="ts">
  import FormField from "./lib/FormField.svelte";

  const checklist = {
    customer: {
      name: "",
      emailAddress: "",
      phoneNumber: "",
    },
    general: {
      visitDate: "",
      reportDate: "",
    },
    coach: {
      name: "",
      emailAddress: "",
      phoneNumber: "",
    },
    building: {
      street: "",
      houseNumber: "",
      postalCode: "",
      city: "",
      constructionYear: null,
    },
  }

  let updateAddressTimeout = null

  function canonicalizePostalCode(postalCode: string): string {
    const match = postalCode.match(/^\s*(\d{4})\s*([a-zA-Z]{2})\s*$/)
    if (match) {
      return `${match[1]} ${match[2].toUpperCase()}`
    } else {
      return postalCode
    }
  }

  function addressChanged() {
    checklist.building.postalCode = canonicalizePostalCode(checklist.building.postalCode)

    let bagViewerSearchQuery = checklist.building.postalCode.replace(' ', '') + ' ' + checklist.building.houseNumber
    bagViewerUrl = `https://bagviewer.kadaster.nl/lvbag/bag-viewer/index.html#`
      + `?searchQuery=${encodeURIComponent(bagViewerSearchQuery)}`
    
    updateAddress()
  }

  async function updateAddress() {
    // https://github.com/PDOK/locatieserver/wiki/API-Locatieserver
    const url = `https://geodata.nationaalgeoregister.nl/locatieserver/v3/free`
      + `?fq=type:adres`
      + `&fq=postcode:${encodeURIComponent(checklist.building.postalCode.replace(' ', ''))}`
      + `&fq=huisnummer:${encodeURIComponent(checklist.building.houseNumber)}`
    const response = await fetch(url)
    if (!response.ok) {
      throw new Error(`Got response code ${response.status}`)
    }
    const json = await response.json()
    if (json.response.docs.length == 0) {
      throw new Error(`Got no results`)
    }

    // First result has the highest score.
    const doc = json.response.docs[0]
    checklist.building.street = doc.straatnaam
    checklist.building.city = doc.woonplaatsnaam
  }

  let bagViewerUrl = null
</script>

<main>
  <h1>Energiecoach</h1>

  <section>
    <h2>Klantgegevens</h2>
    <FormField label="Naam">
      <input type="text" bind:value={checklist.customer.name}>
    </FormField>
    <FormField label="Postcode">
      <input type="text" pattern="^[0-9]{4} [A-Z]{2}$" bind:value={checklist.building.postalCode} on:change={addressChanged}>
    </FormField>
    <FormField label="Huisnummer">
      <input type="text" bind:value={checklist.building.houseNumber} on:change={addressChanged}>
    </FormField>
    <FormField label="Straatnaam">
      <input type="text" bind:value={checklist.building.street}>
    </FormField>
    <FormField label="Plaats">
      <input type="text" bind:value={checklist.building.city}>
    </FormField>
    <FormField label="E-mailadres">
      <input type="email" bind:value={checklist.customer.emailAddress}>
    </FormField>
    <FormField label="Telefoonnummer">
      <input type="text" bind:value={checklist.customer.phoneNumber}>
    </FormField>
  </section>

  <section>
    <h2>Algemeen</h2>
    <FormField label="Datum bezoek">
      <input type="date" bind:value={checklist.general.visitDate}>
    </FormField>
    <FormField label="Datum rapport">
      <input type="date" bind:value={checklist.general.reportDate}>
    </FormField>
    <FormField label="Naam coach">
      <input type="text" bind:value={checklist.coach.name}>
    </FormField>
    <FormField label="E-mailadres coach">
      <input type="text" bind:value={checklist.coach.emailAddress}>
    </FormField>
    <FormField label="Telefoonnummer coach">
      <input type="text" bind:value={checklist.coach.phoneNumber}>
    </FormField>
  </section>

  <section>
    <h2>Woning</h2>
    {#if bagViewerUrl != null}
      <a href={bagViewerUrl} target="_blank" rel="noreferrer">Bekijk pand in BAG Viewer</a>
    {/if}
  </section>
</main>

<style>
  main {
    max-width: 960px;
    margin: 20px auto;
    background-color: white;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
    padding: 20px;
 }
</style>