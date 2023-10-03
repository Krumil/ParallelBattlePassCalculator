<template>
	<div class="main">
		<h1>Games Needed to Reach Level 30</h1>

		<div class="input-row">
			<div class="input-group">
				<InfoLabel forId="startLevel" labelText="Start Level" tooltipText="Your current level in the game." />
				<input id="startLevel" v-model="startLevel" type="number" />
			</div>
			<div class="input-group">
				<InfoLabel forId="levelToReach" labelText="Level to Reach" tooltipText="The level you aim to reach." />
				<input id="levelToReach" v-model="levelToReach" type="number" />
			</div>
			<div class="input-group">
				<InfoLabel
					forId="gameDuration"
					labelText="Game Duration"
					tooltipText="Average duration of each game in minutes." />
				<input id="gameDuration" v-model="gameDuration" type="number" placeholder="e.g., 30" />
			</div>

			<div class="input-group">
				<InfoLabel
					forId="missedMissions"
					labelText="Missed Missions"
					tooltipText="Number of daily missions you will miss." />
				<input id="missedMissions" v-model="missedMissions" type="number" />
			</div>
			<div class="input-group">
				<InfoLabel
					forId="winPercentage"
					labelText="Win Percentage"
					tooltipText="Your predicted win rate percentage." />
				<input id="winPercentage" v-model="winPercentage" type="number" />
			</div>
			<div class="input-group">
				<InfoLabel
					forId="weeksMissing"
					labelText="Weeks Missing"
					tooltipText="Number of weeks mission claims you will miss." />
				<input id="weeksMissing" v-model="weeksMissing" type="number" />
			</div>
		</div>

		<button @click="calculateGamesNeeded">Calculate</button>

		<div class="result-container">
			<transition name="slide-up" @after-enter="showSecondText">
				<div v-if="showFirstText">
					<p>
						To reach level
						<span class="highlight">{{ levelToReach }} </span>, you will need to win
						<span class="highlight">{{ animatedWinsNecessary }} </span>
						games. Given your win percentage, this translates to a total of
						<span class="highlight">{{ animatedGamesNecessary }}</span> games.
					</p>
				</div>
			</transition>

			<transition name="slide-up">
				<div v-if="showSecondText">
					<p>
						To achieve this, you need to play
						<span class="highlight">{{ gamesPerDay }}</span>
						games per day and wil require a total of
						<span class="highlight">{{ totalHours }}</span> hours and
						<span class="highlight">{{ totalMinutes }}</span> minutes.
					</p>
				</div>
			</transition>

			<transition name="slide-up">
				<div v-if="showNoWinNecessaryText">
					<p>
						Congrats! You will reach this level only by completing your daily missions and weekly mission
						claims.
					</p>
				</div>
			</transition>
		</div>
	</div>
</template>

<script>
	import { ref, watch } from "vue";
	import InfoLabel from "@/components/InfoLabel.vue";

	export default {
		name: "App",
		components: {
			InfoLabel
		},
		setup() {
			const startLevel = ref(0); // Default value is 0
			const levelToReach = ref(30); // Default value is 30
			const gameDuration = ref(20); // default value: 30 minutes
			const tooltips = ref({
				startLevel: false,
				levelToReach: false,
				gameDuration: false,
				missedMissions: false,
				winPercentage: false,
				weeksMissing: false
			});

			const missedMissions = ref(0);
			const winPercentage = ref(50);
			const weeksMissing = ref(0);
			const animatedWinsNecessary = ref(0);
			const animatedGamesNecessary = ref(0);
			const gamesNecessary = ref(0);
			const winsNecessary = ref(0);
			const gamesPerDay = ref(0);

			const currentDay = ref(null);

			const showResult = ref(false);
			const showFirstText = ref(false);
			const showSecondText = ref(false);
			const showNoWinNecessaryText = ref(false);

			const showTooltip = field => {
				tooltips.value[field] = true;
			};

			const hideTooltip = field => {
				tooltips.value[field] = false;
			};

			const animateNumber = () => {
				let startGames = 0;
				let startWins = 0;
				const endGames = gamesNecessary.value;
				const endWins = winsNecessary.value;
				const duration = 1000; // 2 seconds
				const incrementGames = endGames / (duration / 10); // Assuming we update every 10ms
				const incrementWins = endWins / (duration / 10); // Assuming we update every 10ms

				showFirstText.value = true;

				const interval = setInterval(() => {
					startGames += incrementGames;
					startWins += incrementWins;

					animatedGamesNecessary.value = Math.round(startGames);
					animatedWinsNecessary.value = Math.round(startWins);

					if (animatedGamesNecessary.value >= endGames && animatedWinsNecessary.value >= endWins) {
						clearInterval(interval);
						animatedGamesNecessary.value = endGames;
						animatedWinsNecessary.value = endWins;
						showSecondText.value = true;
					}
				}, 10);
			};

			const calculateGamesNeeded = () => {
				showFirstText.value = false;
				showSecondText.value = false;
				showNoWinNecessaryText.value = false;

				gamesNecessary.value = 0;
				winsNecessary.value = 0;

				const p = winPercentage.value / 100;
				const monthLastDay = new Date(new Date().getFullYear(), new Date().getMonth() + 1, 0).getDate();
				const remainingDays = monthLastDay - (currentDay.value || new Date().getDate()) + 1;
				const xpDaily = 500 * (monthLastDay - missedMissions.value);
				const xpWeekly = 1500 * (4 - weeksMissing.value);
				let totalXp = xpDaily + xpWeekly + startLevel.value * 1000;
				let xpToReach = levelToReach.value * 1000;
				const xpPerWin = 20; // XP awarded on each win
				let xpFromWinBonuses = 750;

				while (totalXp < xpToReach) {
					winsNecessary.value += 1; // Increment the ref value
					totalXp += xpPerWin;

					if (winsNecessary.value % 25 == 0) {
						totalXp += xpFromWinBonuses;
					}
				}

				gamesNecessary.value = Math.ceil(winsNecessary.value / p);
				gamesPerDay.value = Math.ceil(gamesNecessary.value / remainingDays);
				showResult.value = true;

				if (gamesNecessary.value == 0) {
					showNoWinNecessaryText.value = true;
				} else {
					animateNumber();
				}
			};

			watch(gamesNecessary, newValue => {
				if (newValue) {
					animateNumber();
				}
			});

			return {
				animatedGamesNecessary,
				animatedWinsNecessary,
				currentDay,
				gameDuration,
				gamesNecessary,
				gamesPerDay,
				levelToReach,
				missedMissions,
				showResult,
				showFirstText,
				showSecondText,
				showNoWinNecessaryText,
				startLevel,
				weeksMissing,
				winPercentage,
				winsNecessary,
				showTooltip,
				hideTooltip,
				tooltips,
				calculateGamesNeeded
			};
		},
		computed: {
			highlightedResult() {
				return `To reach level 30, you will need <span class="highlight">${this.gamesNecessary}</span> games based on the provided data.`;
			},
			totalHours() {
				return Math.floor((this.gamesNecessary * this.gameDuration) / 60);
			},
			totalMinutes() {
				return (this.gamesNecessary * this.gameDuration) % 60;
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
		display: inline-flex;
		flex-direction: column;
		align-items: center;
		justify-content: flex-start;
		min-height: 100vh;
		color: #def141;
		animation: fadeIn 1s ease-in-out;
		max-width: 786px;
		padding: 0 20px;
		box-sizing: border-box;
		margin: auto;
		position: relative;

		@media (max-width: 768px) {
			overflow: scroll;
		}

		h1 {
			margin-bottom: 20px;
			font-size: 2em;
		}

		p {
			font-size: 24px;
			line-height: 1.2;
		}

		.brief-explanation {
			margin-bottom: 80px;
			text-align: start;
			font-size: 20px;
		}

		.input-row {
			display: inline-flex;
			justify-content: space-between;
			flex-wrap: wrap; // This allows wrapping to the next line if more than three.
			width: 100%;
			margin-bottom: 20px;

			.input-group {
				text-align: start;
				flex: 0 0 33.33%;
				padding: 0 20px;

				@media (max-width: 768px) {
					padding: 0;
					flex: 0 0 calc(50% - 6px);
				}

				label {
					display: inline-flex;
					margin: 10px 0;
					justify-content: center;
					align-items: center;
				}
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

		.result-container {
			text-align: start;
			margin-top: 20px;
			position: relative;
			padding-bottom: 20px;

			p {
				width: 100%;
			}
		}
	}

	.slide-up-enter-active {
		animation: slideUp 1s forwards;
	}

	.highlight {
		color: #dc143c;
		font-weight: bold;
		font-size: 30px;
	}
</style>
