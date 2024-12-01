<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pokémon Card Game</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f9f9f9;
            text-align: center;
        }

        .container {
            margin-top: 20px;
        }

        .card {
            display: inline-block;
            width: 150px;
            height: 200px;
            border: 2px solid #000;
            margin: 10px;
            padding: 10px;
            background-color: white;
            box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.2);
        }

        .card img {
            width: 100%;
            height: auto;
        }

        .card p {
            margin: 10px 0;
        }

        .money {
            font-size: 20px;
            margin-top: 20px;
        }

        .battle-result {
            font-size: 18px;
            margin-top: 20px;
            font-weight: bold;
        }

        .stats-container {
            display: none;
            text-align: left;
            margin: 20px;
            border: 1px solid #ccc;
            padding: 20px;
            background-color: #fff;
        }

        .button {
            padding: 10px 20px;
            margin: 10px;
            background-color: #008CBA;
            color: white;
            border: none;
            cursor: pointer;
        }

        .button:hover {
            background-color: #006f8a;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Pokémon Card Game</h1>
        <div id="status"></div>
        <button class="button" id="openPackButton">Open a Pack (50 Money)</button>
        <button class="button" id="battleButton">Battle a Random Pokémon</button>
        <button class="button" id="sendToWorkButton">Send Pokémon to Work</button>
        <button class="button" id="tradeButton">Trade Pokémon</button>
        <button class="button" id="showStatsButton">Show Stats</button>
        <div class="money" id="moneyDisplay">Money: 50</div>

        <div id="cardsDisplay"></div>
        <div class="battle-result" id="battleResult"></div>
    </div>

    <div class="stats-container" id="statsContainer">
        <h2>Player Stats</h2>
        <p id="mostBattlesStat"></p>
        <p id="highestLevelStat"></p>
        <p id="mostWorkedStat"></p>
        <p id="mostPowerStat"></p>
        <p id="leastPowerStat"></p>
        <button class="button" id="closeStatsButton">Close Stats</button>
    </div>

    <script>
        const API_URL = 'https://pokeapi.co/api/v2/pokemon?limit=1000';
        let allPokemons = [];
        let money = 50;
        let cards = [];
        let stats = {
            battlesWon: {}, // Tracks battles won by Pokémon
            workCount: {},  // Tracks work tasks done by Pokémon
        };

        const openPackButton = document.getElementById('openPackButton');
        const battleButton = document.getElementById('battleButton');
        const sendToWorkButton = document.getElementById('sendToWorkButton');
        const tradeButton = document.getElementById('tradeButton');
        const showStatsButton = document.getElementById('showStatsButton');
        const moneyDisplay = document.getElementById('moneyDisplay');
        const cardsDisplay = document.getElementById('cardsDisplay');
        const battleResult = document.getElementById('battleResult');
        const statsContainer = document.getElementById('statsContainer');
        const mostBattlesStat = document.getElementById('mostBattlesStat');
        const highestLevelStat = document.getElementById('highestLevelStat');
        const mostWorkedStat = document.getElementById('mostWorkedStat');
        const mostPowerStat = document.getElementById('mostPowerStat');
        const leastPowerStat = document.getElementById('leastPowerStat');
        const closeStatsButton = document.getElementById('closeStatsButton');

        // Fetch Pokémon data
        async function fetchPokemons() {
            try {
                const response = await fetch(API_URL);
                const data = await response.json();
                allPokemons = data.results;
            } catch (error) {
                console.error('Error fetching Pokémon:', error);
            }
        }

        fetchPokemons();

        // Get a random card, including Gojo as a rare Pokémon
        function getRandomCard() {
            // 1 in 100 chance to get "Gojo" from JJK as a rare Pokémon
            const isGojo = Math.random() < 0.01; // 1% chance for Gojo
            if (isGojo) {
                return {
                    name: 'Gojo',
                    power: 999, // Gojo is extremely powerful
                    img: 'https://static.wikia.nocookie.net/jujutsu-kaisen/images/e/ef/Satoru_Gojo_%28Anime_2%29.png/revision/latest?cb=20240622022211', // Corrected Gojo image URL
                    level: 1,
                    exp: 0,
                };
            }

            const randomPokemon = allPokemons[Math.floor(Math.random() * allPokemons.length)];
            return {
                name: randomPokemon.name,
                power: Math.floor(Math.random() * 100) + 50,
                img: `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/${randomPokemon.url.split('/')[6]}.png`,
                level: 1,
                exp: 0,
            };
        }

        // Display cards you own
        function displayCards(cardsList, container) {
            container.innerHTML = '';
            cardsList.forEach(card => {
                const cardElement = document.createElement('div');
                cardElement.classList.add('card');
                cardElement.innerHTML = `
                    <img src="${card.img}" alt="${card.name}">
                    <p>${card.name}</p>
                    <p>Power: ${card.power}</p>
                    <p>Level: ${card.level}</p>
                    <button class="button" onclick="sellCard('${card.name}')">Sell Card</button>
                `;
                container.appendChild(cardElement);
            });
        }

        // Sell a card
        function sellCard(cardName) {
            const cardIndex = cards.findIndex(card => card.name === cardName);
            if (cardIndex !== -1) {
                const card = cards[cardIndex];
                // Earn money based on the Pokémon's power
                const moneyEarned = Math.floor(card.power / 2);
                money += moneyEarned;
                moneyDisplay.textContent = `Money: ${money}`;
                // Remove the card from the player's collection
                cards.splice(cardIndex, 1);
                displayCards(cards, cardsDisplay);
            }
        }

        // Open a pack
        openPackButton.onclick = () => {
            if (money < 50) {
                alert('Not enough money!');
                return;
            }

            money -= 50;
            moneyDisplay.textContent = `Money: ${money}`;

            let newCard = getRandomCard();
            while (cards.some(card => card.name === newCard.name)) {
                newCard = getRandomCard(); // Ensure you don't get duplicates
            }

            cards.push(newCard);
            displayCards(cards, cardsDisplay);
        };

        // Battle a random Pokémon (old battle system with details)
        battleButton.onclick = () => {
            if (cards.length === 0) {
                alert("You have no Pokémon cards to battle with!");
                return;
            }

            // Let the player choose their Pokémon to battle
            const playerCard = cards[Math.floor(Math.random() * cards.length)];

            // Generate a random opponent Pokémon
            const opponentCard = getRandomCard();

            // Track battles won for each Pokémon
            if (!stats.battlesWon[playerCard.name]) stats.battlesWon[playerCard.name] = 0;

            const battleMessage = `
                <p>Your Pokémon: ${playerCard.name} (Power: ${playerCard.power})</p>
                <p>Opponent: ${opponentCard.name} (Power: ${opponentCard.power})</p>
            `;
            battleResult.innerHTML = battleMessage;

            // Battle logic
            if (playerCard.power > opponentCard.power) {
                stats.battlesWon[playerCard.name]++;
                const moneyWon = Math.floor(opponentCard.power / 2);
                money += moneyWon;
                battleResult.innerHTML += `<p>You won! You earned ${moneyWon} money!</p>`;
                // Give the winning Pokémon some experience
                playerCard.exp += 10;
                if (playerCard.exp >= 100) {
                    playerCard.level++;
                    playerCard.exp = 0;
                    battleResult.innerHTML += `<p>${playerCard.name} leveled up to Level ${playerCard.level}!</p>`;
                }
            } else {
                battleResult.innerHTML += `<p>You lost. Better luck next time!</p>`;
            }

            moneyDisplay.textContent = `Money: ${money}`;
        };

        // Send Pokémon to work
        sendToWorkButton.onclick = () => {
            if (cards.length === 0) {
                alert("You have no Pokémon cards to send to work!");
                return;
            }

            const randomCard = cards[Math.floor(Math.random() * cards.length)];
            if (!stats.workCount[randomCard.name]) stats.workCount[randomCard.name] = 0;
            stats.workCount[randomCard.name]++;

            // Simulate work result (money, experience, or nothing)
            const workResult = Math.random();
            if (workResult < 0.5) {
                randomCard.exp += 10;
                alert(`${randomCard.name} earned 10 EXP at work!`);
            } else if (workResult < 0.8) {
                money += 20;
                alert(`${randomCard.name} earned 20 money at work!`);
            } else {
                alert(`${randomCard.name} came back with nothing.`);
            }
        };

        // Show stats
        showStatsButton.onclick = () => {
            statsContainer.style.display = 'block';

            const mostBattles = Object.entries(stats.battlesWon).sort((a, b) => b[1] - a[1])[0];
            const mostWorked = Object.entries(stats.workCount).sort((a, b) => b[1] - a[1])[0];
            const highestLevel = cards.sort((a, b) => b.level - a.level)[0];
            const mostPower = cards.sort((a, b) => b.power - a.power)[0];
            const leastPower = cards.sort((a, b) => a.power - b.power)[0];

            mostBattlesStat.textContent = `Pokémon with most battles won: ${mostBattles ? mostBattles[0] : 'None'} (${mostBattles ? mostBattles[1] : 0} wins)`;
            mostWorkedStat.textContent = `Pokémon that worked the most: ${mostWorked ? mostWorked[0] : 'None'} (${mostWorked ? mostWorked[1] : 0} tasks)`;
            highestLevelStat.textContent = `Highest level Pokémon: ${highestLevel ? highestLevel.name : 'None'} (Level: ${highestLevel ? highestLevel.level : 0})`;
            mostPowerStat.textContent = `Pokémon with the most power: ${mostPower ? mostPower.name : 'None'} (Power: ${mostPower ? mostPower.power : 0})`;
            leastPowerStat.textContent = `Pokémon with the least power: ${leastPower ? leastPower.name : 'None'} (Power: ${leastPower ? leastPower.power : 0})`;
        };

        closeStatsButton.onclick = () => {
            statsContainer.style.display = 'none';
        };
    </script>
</body>
</html>
