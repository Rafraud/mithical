<template>
  <WaccaProfileRequired>
    <v-container class="elevation-1 mt-4 pa-0">
      <div v-if="leaderboardsLoading" class="d-flex justify-center">
        <v-progress-circular
          indeterminate
          color="primary"
          :size="80"
          :width="10"
          class="mt-4"
        ></v-progress-circular>
      </div>

      <div v-else>
          <h3 class="your-rank">
            Your Rank: {{ getRankDescription(profile.user_name) }}
          </h3>
        <v-alert v-if="leaderboardsLoadingError" type="error" class="mt-4">{{
          leaderboardsLoadingError
        }}</v-alert>

        <v-table v-else>
          <thead>
            <tr>
              <th width="1%">Rank</th>
              <th>Name</th>
              <th>Rating</th>
            </tr>
          </thead>

          <tbody>
            <tr 
              v-for="(player, i) in leaderboardData" 
              :key="i"
              :class="{ highlight: player.user_name == profile.user_name }"
              >
              <td class="text-right">
                <span
                  v-if="
                    i == 0 || leaderboardData[i - 1].rating != player.rating
                  "
                >
                  {{ i + 1 }}
                </span>
                <span v-else>=</span>
              </td>
              <td>
                <WaccaIcon
                  class="leaderboard-icon"
                  :icon="player.user_icon_id"
                />
                {{ player.user_name }}
              </td>
              <td>
                <WaccaRating
                  class="rating"
                  :rating="player.rating"
                  :simple="i >= 10"
                />
              </td>
            </tr>
          </tbody>
        </v-table>
      </div>
    </v-container>
  </WaccaProfileRequired>
</template>

<style scoped lang="scss">
.leaderboard-icon {
  height: 40px;
  vertical-align: middle;
}

.rating {
  vertical-align: middle;
  font-size: 2.5em;
}

tr.highlight {
  background-color: rgba(var(--v-theme-primary), 0.1);
}

:deep(.rating-white) {
  color: rgba(var(--v-theme-on-surface), var(--v-high-emphasis-opacity));
}
</style>

<script setup>
definePageMeta({
  middleware: ["auth"],
});

const leaderboardsLoading = ref(false);
const leaderboardsLoadingError = ref();
const highscores = ref([]);
const runtimeConfig = useRuntimeConfig();
const leaderboardData = ref([]);
const version = useState("version");
const profile = useState("profile");


function getRankDescription(username) {
  if (!highscores.value) {
    return "Unknown";
  }

  // If there is more than one user with the same score, the displayed rank is equal to the first listed player with that score
  let playerRank;
  let currentRanking;
  let currentScore;
  for (let i = 0; i < leaderboardData.value.length; i++) {
    if (!currentScore || leaderboardData.value[i].score < currentScore) {
      currentScore = leaderboardData.value[i].score;
      currentRanking = i + 1;
    }
    if (leaderboardData.value[i].user_name === username) {
      playerRank = currentRanking;
      break;
    }
  }

  const scoreCount =
    leaderboardData.value.length === 100 ? "100" : leaderboardData.value.length;

  return !playerRank ? "Unranked" : `${playerRank} / ${scoreCount}`;
}

function loadData() {
  leaderboardsLoading.value = ref(true);

  $fetch(
    `${runtimeConfig.public.apiUrl}/wacca/user/leaderboards/${version.value}`
  )
    .then((data) => {
      leaderboardsLoading.value = false;
      leaderboardData.value = data;
    })
    .catch((err) => {
      leaderboardsLoading.value = false;
      leaderboardsLoadingError.value =
        "Couldn't reach the API. Please try again later.";
    });
}

loadData();
watch(version, loadData);
</script>
