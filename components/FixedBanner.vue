<script>
import { MANAGEMENT } from '@/config/types';
import { SETTING } from '@/config/settings';
import isEmpty from 'lodash/isEmpty';

export default {
  props: {
    header: {
      type:    Boolean,
      default: false
    },
    consent: {
      type:    Boolean,
      default: false
    },
    footer: {
      type:    Boolean,
      default: false
    },
  },

  async fetch() {
    this.bannerSetting = await this.$store.getters['management/byId'](MANAGEMENT.SETTING, SETTING.BANNERS);
  },

  data() {
    return {
      showDialog:      true,
      showHeader:      false,
      showFooter:      false,
      showConsent:     false,
      banner:          {},
      bannerSetting:   null
    };
  },

  methods: {
    hideDialog() {
      this.showDialog = false;
    }
  },

  computed: {
    bannerStyle() {
      return {
        color:              this.banner.color,
        'background-color': this.banner.background,
        'text-align':       this.banner.textAlignment,
        'font-weight':      this.banner.fontWeight ? 'bold' : '',
        'font-style':       this.banner.fontStyle ? 'italic' : '',
        'font-size':        this.banner.fontSize,
        'text-decoration':  this.banner.textDecoration ? 'underline' : ''
      };
    },
    dialogStyle() {
      return {
        color:              this.banner.color,
        'background-color': this.banner.background
      };
    },
    showBanner() {
      if (!this.banner.text && !this.banner.background) {
        return false;
      }

      if (this.header) {
        return this.showHeader;
      } else if (this.consent) {
        return this.showConsent;
      } else if (this.footer) {
        return this.showFooter;
      }

      return null;
    },

    showAsDialog() {
      return this.consent && !!this.banner.button;
    }
  },

  watch: {
    bannerSetting(neu) {
      if (neu?.value && neu.value !== '') {
        try {
          const parsed = JSON.parse(neu.value);
          const {
            bannerHeader, bannerFooter, bannerConsent, banner, showHeader, showFooter, showConsent
          } = parsed;
          let bannerContent = parsed?.banner || {};

          if (isEmpty(banner)) {
            if (showHeader && this.header) {
              bannerContent = bannerHeader || {};
            } else if (showConsent && this.consent) {
              bannerContent = bannerConsent || {};
            } else if (showFooter && this.footer) {
              bannerContent = bannerFooter || {};
            } else {
              bannerContent = {};
            }
          }

          this.showHeader = showHeader === 'true';
          this.showFooter = showFooter === 'true';
          this.showConsent = showConsent === 'true';
          this.banner = bannerContent;
        } catch {}
      }
    }
  }
};
</script>

<template>
  <div v-if="showBanner">
    <div v-if="!showAsDialog" class="banner banner-banner" :style="bannerStyle" :class="{'banner-consent': consent}">
      {{ banner.text }}
    </div>
    <div v-else-if="showDialog">
      <div class="banner-dialog-glass"></div>
      <div class="banner-dialog">
        <div class="banner-dialog-frame" :style="dialogStyle">
          <div class="banner" :style="bannerStyle">
            {{ banner.text }}
          </div>
          <button class="btn role-primary" @click="hideDialog()">
            {{ banner.button }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
  .banner {
    text-align: center;
    line-height: 2em;
    height: 2em;
    width: 100%;
    padding: 0 20px;

    &.banner-consent {
      position: absolute;
      height: unset;
      min-height: 2em;
      max-height: 4em;
      overflow: hidden;
    }
  }
  .banner-dialog, .banner-dialog-glass {
    position: absolute;
    top: 0px;
    left: 0px;
    width: 100vw;
    height: 100vh;
  }
  .banner-dialog-glass {
    z-index: 5000;
    background-color: var(--default);
    opacity: 0.75;
  }
  .banner-banner {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
  .banner-dialog {
    z-index: 5001;
    display: flex;
    align-items: center;
    justify-content: center;

    .banner-dialog-frame {
      border: 2px solid var(--border);
      display: flex;
      align-items: center;
      flex-direction: column;
      padding: 20px;
      height: fit-content;
      width: fit-content;
      min-width: 50%;
      max-width: 80%;
      max-height: 90%;

      .banner {
        height: initial;
        overflow-y: auto;
      }

      button {
        margin-top: 10px;
        max-width: 50%;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
        width: fit-content;
      }
    }
  }
</style>
