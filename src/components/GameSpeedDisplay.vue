<script>
import { getGameSpeedupFactor } from '../game';

export default {
  name: "GameSpeedDisplay",
  props: {
  },
  data() {
    return {
      baseSpeed: new Decimal(),
      baseSpeedPreExpo: new Decimal(),
      pulsedSpeed: new Decimal(),
      hasSeenAlteredSpeed: false,
      isStopped: false,
      isEC12: false,
      isPulsing: false,
      hasBH3: false,
      BH3Power: 1,
      pastGSSoftcap: false,
      scOne: new Decimal(),
      scOneEffect: 1
    };
  },
  computed: {
    baseSpeedText() {
      if (this.isStopped) {
        return "Stopped (storing real time)";
      }
      const speed = this.formatNumber(this.baseSpeed);
      const speedPreExpo = this.formatNumber(Decimal.pow(this.baseSpeed, 1 / this.BH3Power));
      if (this.isEC12) {
        return `${speed} (fixed)`;
      }
      if (this.BH3Power > 1 && this.baseSpeed.gte(1)) return `${speedPreExpo}`;
      return `${speed}`;
    },
    pulseSpeedText() {
      return `${this.formatNumber(this.pulsedSpeed)}`;
    },
    baseText() {
      let x = Decimal.pow(this.baseSpeedPreExpo, this.BH3Power);
      if (!this.hasSeenAlteredSpeed) return null;
      if (this.isStopped) return `Game speed is altered: ${this.baseSpeedText}`
      return this.baseSpeed.eq(1)
        ? "The game is running at normal speed."
        : this.hasBH3 && this.baseSpeed.gte(1) && true ? `Game speed is altered: ${format(x, 2, 2)} (${this.baseSpeedText}${formatPow(this.BH3Power, 3, 3)})`: `Game speed is altered: ${this.baseSpeedText}`;
    }
  },
  methods: {
    update() {
      this.baseSpeed.copyFrom(getGameSpeedupFactor());
      this.baseSpeedPreExpo = Decimal.pow(this.baseSpeed, 1 / this.BH3Power);
      this.pulsedSpeed.copyFrom(getGameSpeedupForDisplay());
      this.hasSeenAlteredSpeed = PlayerProgress.seenAlteredSpeed();
      this.isStopped = Enslaved.isStoringRealTime;
      this.isEC12 = EternityChallenge(12).isRunning;
      this.isPulsing = (this.baseSpeed.neq(this.pulsedSpeed)) && Enslaved.canRelease(true);
      this.hasBH3 = ExpoBlackHole(1).isUnlocked;
      this.BH3Power = ExpoBlackHole(1).power;
      this.pastGSSoftcap = getGameSpeedupFactor().gte(this.scOneStart);
      this.scOneStart = getGameSpeedupSoftcaps();
      this.scOneEffect = 0.4321;
    },
    formatNumber(num) {
      if (num.gte(0.001) && num.lt(10000) && num.neq(1)) {
        return format(num, 3, 3);
      }
      if (num.lt(0.001)) {
        return `${formatInt(1)} / ${format(new Decimal(1).div(num), 2)}`;
      }
      return `${format(num, 2)}`;
    }
  }
};
</script>

<template>
  <span class="c-gamespeed">
    <span>
      {{ baseText }}
    </span>
    <span v-if="isPulsing">(<i class="fas fa-expand-arrows-alt u-fa-padding" /> {{ pulseSpeedText }})</span>
    <span v-if="pastGSSoftcap" class="c-gssoftcapone">
      <br>
      Due to instability, Game Speed past {{ format(scOneStart) }} is raised {{formatPow(scOneEffect, 3, 3) }} after all other Game Speed effects
  </span>
  </span>
</template>

<style scoped>
.c-gamespeed {
  font-weight: bold;
  color: var(--color-text);
}

.c-gssoftcapone {
  font-weight: bold;
  color: #FF0000;
}
</style>
