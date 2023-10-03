<template>
	<div class="main">
		<h1>Games Needed to Reach Level 30</h1>

		<div class="input-row">
			<div class="input-group">
				<label for="startLevel">Start Level:</label>
				<input id="startLevel" v-model="startLevel" type="number" />
			</div>
			<div class="input-group">
				<label for="levelToReach">Level to Reach:</label>
				<input id="levelToReach" v-model="levelToReach" type="number" />
			</div>
			<div class="input-group">
				<label for="gameDuration">Game Duration (minutes):</label>
				<input id="gameDuration" v-model="gameDuration" type="number" placeholder="e.g., 30" />
			</div>
		</div>

		<div class="input-row">
			<div class="input-group">
				<label for="missedMissions">Missed Missions:</label>
				<input id="missedMissions" v-model="missedMissions" type="number" />
			</div>
			<div class="input-group">
				<label for="winPercentage">Win Percentage:</label>
				<input id="winPercentage" v-model="winPercentage" type="number" />
			</div>
			<div class="input-group">
				<label for="weeksMissing">Weeks Missing:</label>
				<input id="weeksMissing" v-model="weeksMissing" type="number" />
			</div>
		</div>

		<button @click="calculateGamesNeeded">Calculate</button>

		<!-- <h1>Games Needed to Reach Level 30</h1>

		<div class="input-row">
			<div class="input-group">
				<label for="levelMissing">Weeks Missing:</label>
				<input id="levelMissing" v-model="weeksMissing" type="number" />
			</div>
		</div>
		<button @click="calculateGamesNeeded">Calculate</button> -->

		<transition name="slide-up">
			<div v-if="showResult">
				<p>
					To reach level 30, you will need <span class="highlight">{{ animatedGamesNecessary }}</span> games
					based on the provided data.
				</p>
				<p>
					To achieve this, you need to play <span class="highlight">{{ gamesPerDay }}</span> games per day.
				</p>
			</div>
		</transition>
	</div>
</template>

<script>
import { ref, watch } from "vue";

export default {
	name: "App",
	setup() {
		const startLevel = ref(0); // Default value is 0
		const levelToReach = ref(30); // Default value is 30
		const gameDuration = ref(20); // default value: 30 minutes

		const missedMissions = ref(0);
		const winPercentage = ref(50);
		const weeksMissing = ref(0);
		const animatedGamesNecessary = ref(0);
		const gamesPerDay = ref(0);

		const currentDay = ref(null);
		const gamesNecessary = ref(null);

		const showResult = ref(false);

		const animateNumber = () => {
			let start = 0;
			const end = gamesNecessary.value;
			const duration = 2000; // 2 seconds
			const increment = end / (duration / 10); // Assuming we update every 10ms

			const interval = setInterval(() => {
				start += increment;
				animatedGamesNecessary.value = Math.round(start);

				if (animatedGamesNecessary.value >= end) {
					clearInterval(interval);
					animatedGamesNecessary.value = end;
				}
			}, 10);
		};

		const calculateGamesNeeded = () => {
			const p = winPercentage.value / 100;
			const monthLastDay = new Date(new Date().getFullYear(), new Date().getMonth() + 1, 0).getDate();
			const remainingDays = monthLastDay - (currentDay.value || new Date().getDate()) + 1;
			const xpDaily = 500 * (monthLastDay - missedMissions.value);
			const xpWeekly = 1250 * (4 - weeksMissing.value);
			let totalXp = xpDaily + xpWeekly + startLevel.value * 1000;
			let xpToReach = levelToReach.value * 1000;
			const xpPerWin = 20; // XP awarded on each win
			let xpFromWinBonuses = 1250;
			let winsNecessary = 0;

			while (totalXp < xpToReach) {
				winsNecessary += 1;
				totalXp += xpPerWin;

				if (winsNecessary % 25 == 0) {
					totalXp += xpFromWinBonuses;
				}
			}

			gamesNecessary.value = Math.ceil(winsNecessary / p);
			gamesPerDay.value = Math.ceil(gamesNecessary.value / remainingDays);
			showResult.value = true;

			animateNumber();
		};

		watch(gamesNecessary, newValue => {
			if (newValue) {
				animateNumber();
			}
		});

		return {
			animatedGamesNecessary,
			currentDay,
			gameDuration,
			gamesNecessary,
			gamesPerDay,
			levelToReach,
			missedMissions,
			showResult,
			startLevel,
			weeksMissing,
			winPercentage,
			calculateGamesNeeded
		};
	},
	computed: {
		highlightedResult() {
			return `To reach level 30, you will need <span class="highlight">${this.gamesNecessary}</span> games based on the provided data.`;
		}
	}
};
</script>

<style lang="scss" scoped>
@keyframes fadeIn {
	from {
		opacity: 0;
	}
	to {
		opacity: 1;
	}
}
@keyframes slideUp {
	from {
		transform: translateY(100%);
		opacity: 0;
	}
	to {
		transform: translateY(0);
		opacity: 1;
	}
}

.main {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	height: 100vh;
	color: #def141;
	animation: fadeIn 1s ease-in-out;
	max-width: 786px;
	padding: 0 20px;
	box-sizing: border-box;
	margin: auto;

	h1 {
		margin-bottom: 20px;
		font-size: 2em;
	}

	.input-row {
		display: flex;
		justify-content: space-between;
		flex-wrap: wrap; // This allows wrapping to the next line if more than three.
		width: 100%;
		margin-bottom: 20px;

		@media (max-width: 768px) {
			flex-direction: column;
		}
	}

	.input-group {
		flex: 0 0 calc(33.3333% - 40px); // Giving it a third of the width minus margins
		margin: 0 20px;

		label {
			display: block;
			margin-bottom: 10px;
		}
	}

	input {
		width: 100%;
		padding: 10px;
		border: none;
		border-radius: 5px;
		background-color: rgba(255, 255, 255, 0.8);
		color: black;
		outline: none;
		transition: background-color 0.3s ease, transform 0.3s ease;
	}

	input:focus {
		background-color: rgba(255, 255, 255, 0.8);
		transform: scale(1.05);
	}

	button {
		padding: 10px 20px;
		border: none;
		border-radius: 5px;
		background-color: black;
		border: 2px solid #def141;
		color: #def141;
		cursor: pointer;
		transition: background-color 0.3s ease, transform 0.2s ease;
		width: 100%;
	}

	button:hover {
		background-color: #203a43;
	}

	button:active {
		transform: scale(0.95);
	}

	h2 {
		margin-top: 20px;
	}

	.result {
		margin-top: 20px;
		font-size: 2.5em;
		text-align: center;
		line-height: 1.5em;
	}
}

.typewriter-enter-active {
	animation: typewriter 2s steps(40, end) forwards;
	overflow: hidden;
	white-space: nowrap;
	border-right: 0.15em solid black; // This creates the blinking cursor effect
}

.slide-up-enter-active {
	animation: slideUp 1s forwards;
}

.highlight {
	color: #def141;
	font-weight: bold;
}
</style>
