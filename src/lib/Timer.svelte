<script>
import TimerDisplay from "$lib/TimerDisplay.svelte";
import TimerControls from "$lib/TimerControls.svelte";
import { setInterval, clearInterval } from "worker-timers";
import { db } from "$src/db.js";
import { setContext, onMount } from "svelte";
import { writable } from "svelte/store";
import { liveQuery } from "dexie";
import { NAMES, UNIT, VOLUME_MULTIPLICATOR } from "$src/utils.js";
import soundPath from "$assets/finished.ogg";

let settings;

$: $settings, updateSettings();

const rounds = writable(0);
const state = writable(NAMES.states.focus);
const time = writable(0);
const timeTotal = writable(0);
const timer = writable(null);
const round = writable(1);

const sound = new Audio(soundPath);

const updateSettings = () => {
	if ($settings) {
		if ($rounds != $settings[2].value) {
			rounds.update((rounds) => $settings[2].value);
			if ($round > $rounds) {
				round.update((round) => $rounds);
			}
		}

		if ($timer == null) {
			updateTime();
		}
	}
};

const updateTime = () => {
	time.update((time) => $settings.find((setting) => setting.name === $state).value);
	timeTotal.update((timeTotal) => $time);
};

onMount(() => {
	settings = liveQuery(() => db[NAMES.dbs.time].toArray());
});

const ctx = {
	state,
	time,
	timeTotal,
	timer,
	round,
	rounds,
	startTimer() {
		timer.update((timer) => setInterval(everySecond, UNIT));
	},
	stopTimer() {
		clearInterval($timer);

		timer.update((timer) => null);
	},
	resetTimer() {
		if ($timer != null) {
			this.stopTimer();
		}
		updateTime();
	},
	async skipRound() {
		if ($round >= $rounds) {
			if ($state === NAMES.states.breakLong) {
				round.update((round) => 1);
				state.update((state) => NAMES.states.focus);
			} else {
				state.update((state) => NAMES.states.breakLong);
			}
		} else {
			if ($state === NAMES.states.focus) {
				state.update((state) => NAMES.states.breakShort);
			} else {
				state.update((state) => NAMES.states.focus);
				round.update(($round) => ($round += 1));
			}
		}

		this.resetTimer();

		const autoStart = await db[NAMES.dbs.general].get(NAMES.settings.autoStart);

		if (autoStart.value) {
			this.startTimer();
		}
	}
};

setContext("pomo_timer", ctx);

const everySecond = async () => {
	if ($time > 0) {
		time.update(($time) => $time - UNIT);
	} else {
		const playSound = await db[NAMES.dbs.general].get(NAMES.settings.playSounds);

		if (playSound.value) {
			const volumeSetting = await db[NAMES.dbs.general].get(NAMES.settings.volume);

			sound.volume = volumeSetting.value * VOLUME_MULTIPLICATOR;
			sound.play();
		}

		ctx.skipRound();
		updateTime();
	}
};
</script>

<!--------markup-------->

<div class="wrap">
	<TimerDisplay />
	<TimerControls />
</div>

<!--------style-------->
<style lang="sass">
	@use 'src/vars'
	@use 'src/oc'

.wrap
	display: flex
	flex-direction: column
	align-items: center
	justify-content: center
	height: 100%
	padding: vars.$size-2
</style>
