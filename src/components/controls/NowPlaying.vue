<template>
  <div class="now-playing">
    <div class="info">
      <img v-if="nowPlayingItem && nowPlayingItem.attributes.artwork"
          class="artwork"
          :src="nowPlayingItem.attributes.artwork | formatArtworkURL(60)" />
      <img v-else
          class="artwork"
          src="../../assets/icon.svg" />

      <div class="track-info" v-if="nowPlayingItem">
        <p class="name h5 mb-1">{{ nowPlayingItem.attributes.name }}</p>
        <p class="artist text-muted">{{ nowPlayingItem.attributes.artistName }} &mdash; {{ nowPlayingItem.attributes.albumName }}</p>
      </div>
      <div class="track-info" v-else>
        <p class="name h5 mb-1">Hello!</p>
        <p class="artist text-muted">Select any item to play.</p>
      </div>

      <div class="time" v-if="nowPlayingItem">
        <p class="text-muted">{{ playbackTime.currentPlaybackTime | formatSeconds }} / {{ playbackTime.currentPlaybackDuration | formatSeconds }}</p>
      </div>
    </div>

    <div ref="progressTooltip" class="progressTooltip">{{ hoverTooltipTime | formatSeconds }}</div>
    <b-progress ref="songProgress"
      variant="success"
      class="songProgress"
      height="4px"
      v-if="playbackTime"
      @click.native="seekToTime($event)"
      @mousemove.native="getHoverTime($event)"
      @mouseover.native="showTooltip()"
      @mouseleave.native="hideTooltip()">
      <b-progress-bar variant="danger" :value="playbackTime.currentPlaybackTime / playbackTime.currentPlaybackDuration * 100" />
      <b-progress-bar variant="dark" :value="bufferedProgress - playbackTime.currentPlaybackTime / playbackTime.currentPlaybackDuration * 100" />
    </b-progress>
    <b-progress v-else height="4px" :value="0" />
  </div>
</template>

<script>
import { mapState } from 'vuex';

import { formatSeconds, formatArtworkURL } from '../../utils';

export default {
  name: 'NowPlaying',
  filters: {
    formatSeconds,
    formatArtworkURL
  },
  data () {
    return {
      hoverTooltipTime: '0:00'
    };
  },
  computed: {
    ...mapState({
      nowPlayingItem: state => state.musicKit.nowPlayingItem,
      playbackTime: state => state.musicKit.playbackTime,
      bufferedProgress: state => state.musicKit.bufferedProgress
    })
  },
  methods: {
    showTooltip () {
      this.$refs.progressTooltip.style.opacity = 1;
    },
    hideTooltip () {
      this.$refs.progressTooltip.style.opacity = 0;
    },
    getHoverTime (event) {
      var hoverLeftOffset = (event.pageX - this.$refs.songProgress.$el.offsetParent.offsetLeft);
      var percentage = (hoverLeftOffset / this.$refs.songProgress.$el.offsetWidth);
      this.hoverTooltipTime = this.playbackTime.currentPlaybackDuration * percentage;
      var newLeft = event.pageX - this.$refs.songProgress.$el.offsetParent.offsetLeft - (this.$refs.progressTooltip.offsetWidth / 2);
      // do not allow the tooltip to go off the screen, maximum of 10px past progress bar
      if (newLeft > (this.$refs.songProgress.$el.offsetWidth - this.$refs.progressTooltip.offsetWidth + 10)) {
        this.$refs.progressTooltip.style.left = 'auto';
        this.$refs.progressTooltip.style.right = '-10px';
      } else {
        this.$refs.progressTooltip.style.left = newLeft + 'px';
        this.$refs.progressTooltip.style.right = 'auto';
      }
    },
    seekToTime: function (event) {
      const instance = window.MusicKit.getInstance();

      var clickLeftOffset = (event.pageX - event.target.offsetParent.offsetLeft);
      var fullWidth = this.$refs.songProgress.$el.offsetWidth;
      var percentage = (clickLeftOffset / fullWidth);

      instance.player.seekToTime(this.playbackTime.currentPlaybackDuration * percentage);
    }
  }
};
</script>

<style lang="scss" scoped>
p {
  margin: 0;
  padding: 0;
}

.now-playing {
  position: relative;
  overflow: hidden;
  font-size: 0.8rem;
}

.info {
  display: flex;
  align-items: center;
}

.artwork {
  width: 60px;
  height: 60px;
  margin-right: 20px;
}

.track-info {
  display: flex;
  flex-direction: column;
  flex: 2;
  min-width: 0;

  p {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
}

.time {
  flex: 0;
  min-width: 80px;
  margin-right: 20px;
  text-align: right;
}

.songProgress {
  cursor: pointer;
}
.progressTooltip {
  position: absolute;
  bottom: 8px;
  padding: 5px 7px;
  min-width: 45px;
  text-align: center;
  color: #fff;
  border-radius: 2px;
  background: rgba(98, 107, 115, 0.9);
  opacity: 0;
  -webkit-transition: opacity 0.2s;
  transition: opacity 0.2s;
}
</style>