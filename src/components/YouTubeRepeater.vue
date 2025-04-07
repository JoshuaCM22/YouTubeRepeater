<template>
  <div class="youtube-repeater">
    <h1 class="title">
      <img alt="Vue logo" src="../assets/logo.png" class="logo" />
      YouTube Repeater App
    </h1>

    <!-- Input for YouTube Video URL -->
    <input
      ref="videoInput"
      v-model="videoUrl"
      @keydown.enter="loadVideo"
      placeholder="Enter YouTube Video URL"
      class="input-url"
    />
    <button
      @click="loadVideo"
      :disabled="!isYouTubeAPIReady"
      class="btn load-btn"
    >
      Load Video
    </button>

    <!-- YouTube Player Embed -->
    <div id="player"></div>

    <!-- Play/Pause and Repeat Controls -->
    <div v-if="player" class="controls">
      <button v-if="!isPlaying" @click="togglePlayPause" class="btn play-btn">
        Play
      </button>
      <button v-if="isPlaying" @click="togglePlayPause" class="btn pause-btn">
        Pause
      </button>
      <button @click="toggleRepeat" class="btn repeat-btn">
        {{ isRepeating ? "Stop Repeat" : "Repeat Video" }}
      </button>
    </div>

    <!-- Display Repeat Count -->
    <div v-if="isRepeating" class="repeat-count">
      <p>Repeat Count: {{ repeatCount }}</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      videoUrl: "", // Store the full YouTube URL here
      player: null,
      isRepeating: true, // Repeat mode is active by default
      isPlaying: false, // Track the current play state
      repeatCount: 0, // Counter to track the number of repeats
      isYouTubeAPIReady: false, // Flag to track API load status
    };
  },
  mounted() {
    this.changeTitlePage();
    this.loadYouTubeAPI();
    this.focusTextBox();
  },
  methods: {
    changeTitlePage() {
      document.title = "YouTube Repeater App";
    },
    focusTextBox() {
      this.$refs.videoInput.focus();
    },
    loadYouTubeAPI() {
      if (!window.YT) {
        const tag = document.createElement("script");
        tag.src = "https://www.youtube.com/iframe_api";
        const firstScriptTag = document.getElementsByTagName("script")[0];
        firstScriptTag.parentNode.insertBefore(tag, firstScriptTag);

        // Define the global function that YouTube API calls once it’s ready
        window.onYouTubeIframeAPIReady = () => {
          this.isYouTubeAPIReady = true;
        };
      } else {
        // If YT already exists, set the flag immediately
        this.isYouTubeAPIReady = true;
      }
    },
    extractVideoId(url) {
      const regExp =
        /^.*(?:youtu.be\/|v\/|u\/\w\/|embed\/|watch\?v=|&v=)([^#&?]*).*/;
      const match = url.match(regExp);
      return match && match[1].length === 11 ? match[1] : null;
    },
    loadVideo() {
      this.repeatCount = 0;
      const videoId = this.extractVideoId(this.videoUrl);
      if (videoId && this.isYouTubeAPIReady) {
        // If a player already exists, destroy it to override with the new video
        if (this.player) {
          this.player.destroy();
          this.player = null; // Reset the player reference
        }

        // Create a new player with the specified video ID
        this.player = new window.YT.Player("player", {
          height: "360",
          width: "640",
          videoId: videoId,
          events: {
            onReady: this.playVideo, // Auto-play the video once it’s ready
            onStateChange: this.onPlayerStateChange,
          },
        });
        return;
      }

      alert("Invalid YouTube URL");
      this.focusTextBox();
    },
    playVideo() {
      if (this.player) {
        this.player.playVideo();
        this.isPlaying = true;
      }
    },
    togglePlayPause() {
      if (this.isPlaying) {
        this.player.pauseVideo();
      } else {
        this.playVideo();
      }
      this.isPlaying = !this.isPlaying;
    },
    toggleRepeat() {
      this.isRepeating = !this.isRepeating;
      this.repeatCount = 0; // Reset the repeat counter whenever repeat mode is toggled
    },
    onPlayerStateChange(event) {
      if (event.data === window.YT.PlayerState.ENDED) {
        this.isPlaying = false; // Hide the pause button when video ends
        if (this.isRepeating) {
          this.repeatCount++; // Increment repeat count each time video restarts
          this.player.seekTo(0); // Go back to the beginning of the video
          this.player.playVideo();
        }
      } else if (event.data === window.YT.PlayerState.PLAYING) {
        this.isPlaying = true;
      } else if (event.data === window.YT.PlayerState.PAUSED) {
        this.isPlaying = false;
      }
    },
  },
};
</script>

<style scoped>
.youtube-repeater {
  text-align: center;
  background-color: #f0f0f0;
  padding: 20px;
  border-radius: 8px;
  max-width: 1000px;
  margin: 0 auto;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}
#player {
  margin-top: 20px;
}
h1 {
  font-size: 1.8em;
  color: #333;
}
.input-url {
  padding: 8px;
  font-size: 1em;
  width: 70%;
  border: 1px solid #ccc;
  border-radius: 4px;
  margin-bottom: 10px;
}
.btn {
  padding: 8px 15px;
  font-size: 1em;
  border: none;
  border-radius: 4px;
  margin: 5px;
  cursor: pointer;
  color: white;
}
.load-btn {
  background-color: #0073e6;
}
.play-btn {
  background-color: #28a745;
}
.pause-btn {
  background-color: #dc3545;
}
.repeat-btn {
  background-color: #000;
}
.repeat-count {
  margin-top: 15px;
  font-size: 1.1em;
  color: #333;
}
.title {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  text-align: center;
}
.logo {
  width: 60px;
  height: 50px;
}
</style>
