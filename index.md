---
title: Simpsons Episode Finder
author: Javier Millan Acosta
layout: default
---

# Find a Simpsons Episode Released Around Your Birthday
Amaze everybody at the cocktail party by unpromptly bringing up which Simpsons episode came out closest to your birthdate.

<div>
  <label for="day">Day:</label>
  <input type="number" id="day" min="1" max="31" placeholder="DD">

  <label for="month">Month:</label>
  <input type="number" id="month" min="1" max="12" placeholder="MM">

  <label for="year">Year:</label>
  <input type="number" id="year" min="1900" placeholder="YYYY">

  <button onclick="runQuery()">Find Episode</button>
</div>

<div id="result" style="display: none;">
  <!-- Results will be rendered here -->
</div>

<style>
  body {
    font-family: 'Trebuchet MS', sans-serif;
    background-color: #FFEE58; /* Simpsons yellow */
    color: #D32F2F; /* Simpsons red */
    text-align: center;
    padding: 20px;
  }

  label {
    font-weight: bold;
    margin: 0 5px;
  }

  input {
    margin: 5px;
    padding: 5px;
    border: 2px solid #D32F2F;
    border-radius: 5px;
  }

  button {
    background-color: #D32F2F;
    color: white;
    border: none;
    padding: 10px 20px;
    margin-top: 10px;
    border-radius: 5px;
    cursor: pointer;
    font-size: 16px;
  }

  button:hover {
    background-color: #B71C1C;
  }

  #result {
    margin-top: 20px;
    padding: 15px;
    background-color: #FFF8E1; /* Light yellow */
    border: 2px solid #D32F2F;
    border-radius: 10px;
    display: inline-block;
  }

  p {
    margin: 10px 0;
  }
</style>

<script>
async function fetchClosestEpisode(userDate) {
    const endpointUrl = 'https://query.wikidata.org/sparql';
    const date = new Date(userDate);
    const year = date.getFullYear();
    const month = ("0" + (date.getMonth() + 1)).slice(-2); // Ensure month format is two digits

    const query = `
    SELECT DISTINCT ?nameSpanish ?nameEnglish ?seasonNumber ?releasedate WHERE {
      ?episode wdt:P179 wd:Q886 .
      ?episode wdt:P4908 ?season .
      ?episode wdt:P577 ?releasedate .
      ?episode rdfs:label ?nameSpanish .
      FILTER(LANG(?nameSpanish) = "es") .
      ?episode rdfs:label ?nameEnglish .
      FILTER(LANG(?nameEnglish) = "en") .
      ?season rdfs:label ?seasonNumber .
      FILTER(LANG(?seasonNumber) = "en") .
      FILTER(YEAR(?releasedate) = ${year} && MONTH(?releasedate) = ${month})
    } ORDER BY ASC(?releasedate)
    `;

    try {
        const response = await fetch(endpointUrl, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/sparql-query',
                'Accept': 'application/json'
            },
            body: query
        });
        const data = await response.json();
        const episodes = data.results.bindings;

        if (episodes.length === 0) {
            document.getElementById('result').innerText = 'No episodes found for the given month and year.';
            document.getElementById('result').style.display = 'block';
            return;
        }

        // Find the closest episode
        const closestEpisode = episodes.reduce((closest, current) => {
            const currentReleaseDate = new Date(current.releasedate.value);
            return currentReleaseDate < date ? current : closest;
        }, episodes[0]);

        // Format release date to exclude the time
        const formattedReleaseDate = closestEpisode.releasedate.value.split('T')[0];

        document.getElementById('result').innerHTML = `
            <p>This Simpsons episode was released around your birthday:</p>
            <p><strong>${closestEpisode.nameEnglish.value}</strong> (${closestEpisode.nameSpanish.value})</p>
            <p>Release Date: ${formattedReleaseDate}</p>
        `;
        document.getElementById('result').style.display = 'block';
    } catch (error) {
        console.error('Error fetching data:', error);
        document.getElementById('result').innerText = 'Error fetching data. Please try again later.';
        document.getElementById('result').style.display = 'block';
    }
}

function runQuery() {
    const day = document.getElementById('day').value;
    const month = document.getElementById('month').value;
    const year = document.getElementById('year').value;

    if (day && month && year) {
        const formattedDate = `${year}-${("0" + month).slice(-2)}-${("0" + day).slice(-2)}`;
        fetchClosestEpisode(formattedDate);
    } else {
        document.getElementById('result').innerText = 'Please enter a valid date.';
        document.getElementById('result').style.display = 'block';
    }
}
</script>