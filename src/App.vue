<template>
  <header>
    <div class="header-bg">
      <div class="header-bar">
        <h1 class="page-title">dev articles</h1>
      </div>
      <div class="filter-cont">
        <form class="filter-form" action="" method="post">
          <div class="search">
            <button class="search-btn" type="button">
              <img src="./assets/heart-icon.svg" alt="search icon" />
            </button>

            <label class="nolabel" for="filter-1"
              >Filter by title, author</label
            >
            <input
              v-model="searchText"
              type="text"
              name="filter"
              id="filter-1"
              placeholder="Filter by title, author ..."
            />
          </div>
          <select v-model="sortOption">
            <option value="author">Sort by Author</option>
            <option value="date">Sort by Date</option>
          </select>

          <div class="latest">
            <input
              v-model="showOnlyCurrentYear"
              type="checkbox"
              name="latest"
              id="latest"
            />
            <label for="latest">latest only</label>
          </div>
        </form>
      </div>
    </div>
  </header>

  <main class="container">
    <div class="card" v-for="card in filteredAndSortedCards" :key="card.id">
      <div>
        <img
          class="cover"
          :src="card.images[0].landscape[0]"
          alt="article cover image"
        />
        <div class="wrap">
          <div class="info">
            <span class="abbr">{{ getAuthorInitials(card.author) }}</span>
            <div>
              <span class="author">{{ card.author }}</span>
              <span class="date">{{ formatDate(card.dateAdded) }}</span>
            </div>
          </div>
          <h2 class="title">{{ card.title }}</h2>
        </div>
      </div>
      <div class="likes">
        <button
          @click="toggleLike(card)"
          :class="{ 'liked-btn-active': card.liked }"
          class="like-btn"
        >
          Like
        </button>
        <div>
          <img class="lk-icon" src="./assets/heart-icon.svg" alt="heart icon" />
          <span>{{ card.likes }} Likes</span>
        </div>
      </div>
    </div>
  </main>

  <footer>
    <span>Copyright &copy; Younes Cherkaoui</span> <span>2023</span>
  </footer>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      cards: [], // The fetched card data will be stored here
      searchText: "",
      sortOption: "author",
      showOnlyCurrentYear: false,
    };
  },

  methods: {
    fetchData() {
      // Replace the URL with the actual API endpoint URL
      // const apiUrl = "https://myposter.de/web-api/job-interview";
      const apiFile = "/data/api.json";

      fetch(apiFile)
        .then((response) => response.json())
        .then((resData) => {
          // Assuming the API response has an array of cards in 'data.payload.data'

          const {
            payload: { data },
          } = resData;

          this.cards = data;
          console.log(data);
        })
        .catch((error) => {
          console.error("Error fetching data:", error);
        });
    },

    getAuthorInitials(name) {
      return name
        .split(" ")
        .map((part) => part.charAt(0))
        .join("")
        .toUpperCase();
    },

    formatDate(dateString) {
      const date = new Date(dateString);
      return date.toLocaleDateString("en-US");
    },

    toggleLike(card) {
      card.likes += card.liked ? -1 : +1;

      this.cards = this.cards.map((myCard) => {
        if (card.id == myCard.id) {
          return {
            ...myCard,
            liked: !myCard.liked,
          };
        }

        return myCard;
      });
    },
  },

  computed: {
    filteredAndSortedCards() {
      // Filter the cards based on searchText
      let filteredCards = this.cards;
      if (this.searchText) {
        const searchTerm = this.searchText.toLowerCase();
        filteredCards = this.cards.filter(
          (card) =>
            card.author.toLowerCase().includes(searchTerm) ||
            card.title.toLowerCase().includes(searchTerm)
        );
      }

      // Sort the cards based on sortOption
      if (this.sortOption === "author") {
        filteredCards.sort((a, b) => a.author.localeCompare(b.author));
      } else if (this.sortOption === "date") {
        filteredCards.sort(
          (a, b) => new Date(b.dateAdded) - new Date(a.dateAdded)
        );
      }

      // Filter the cards for current year if showOnlyCurrentYear is true
      if (this.showOnlyCurrentYear) {
        const currentYear = new Date().getFullYear();
        filteredCards = filteredCards.filter(
          (card) => new Date(card.dateAdded).getFullYear() === currentYear
        );
      }

      return filteredCards;
    },
  },
  mounted() {
    // Fetch data when the Vue instance is created
    this.fetchData();
  },
};
</script>

<style>
@import "./assets/css/style.css";
</style>
