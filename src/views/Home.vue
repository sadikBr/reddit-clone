<template>
  <div class="home">
    <form @submit.prevent="setSearchTerm" class="form">
      <input v-model="input" type="text" name="search-input" />
      <button type="submit">Search</button>
    </form>

    <div class="results">
      <img
        class="loading-image"
        v-if="loading"
        src="https://i.pinimg.com/originals/f9/41/ae/f941ae9d16fd7d2957eea6e5b1100d1e.gif"
        alt="loading spiner"
      />
      <h3 v-if="error">{{ searchTerm }} NOT FOUND</h3>
      <div
        :key="item.data.title + item.data.created"
        v-for="item in output"
        class="card"
      >
        <router-link
          :to="{
            name: 'Item',
            params: { name: item.data.title, url: item.data.url },
          }"
        >
          <img :src="item.data.url" :alt="item.data.title" />
        </router-link>
        <h1 class="title">
          {{ item.data.title }}
          <span class="score">{{ item.data.score }}</span>
        </h1>
        <div class="footer">
          <p class="date-author">
            created {{ timeago.format(item.data.created_utc * 1000) }} by
            <a
              class="author"
              :href="'https://www.reddit.com/u/' + item.data.author"
              >u/{{ item.data.author }}.
            </a>
          </p>
          <div class="heart"></div>
        </div>
      </div>
    </div>

    <button @click="scrollUp" v-if="fadeIn" class="scroll-up">⬆</button>
  </div>
</template>

<script>
import * as timeago from 'timeago.js';

export default {
  name: 'Home',
  data() {
    return {
      input: '',
      searchTerm: '',
      after: undefined,
      output: [],
      loading: false,
      error: false,
      fadeIn: false,
      callingAPI: false,
      timeago,

      setSearchTerm() {
        this.searchTerm = this.input
          .trim()
          .split(' ')
          .join('');
        this.input = '';
      },
    };
  },
  methods: {
    scrollUp() {
      window.scrollTo({
        top: 0,
        left: 0,
        behavior: 'smooth',
      });
    },
    listenForScrolling() {
      window.addEventListener('scroll', () => {
        this.fadeIn = window.scrollY > 1500;
        if (
          this.searchTerm &&
          document.querySelector('.results').clientHeight - window.scrollY <
            1000 &&
          this.after !== null &&
          !this.callingAPI
        ) {
          this.getData(this.searchTerm, this.after);
        }
      });
    },
    async getData(searchTerm, after) {
      const resultData = [];
      this.callingAPI = true;

      try {
        const response = await fetch(
          `https://www.reddit.com/r/${searchTerm}/.json?limit=50&after=${after}`
        );

        const data = await response.json();

        this.after = data.data.after;

        resultData.push(
          ...data.data.children.filter((item) =>
            item.data.url.match(/(.jpe?g|.png|.gif)$/)
          )
        );
      } catch (error) {
        this.loading = false;
        this.error = true;
        this.callingAPI = false;
      }

      this.loading = false;
      this.callingAPI = false;
      this.output.push(...resultData);
    },
  },
  mounted() {
    this.listenForScrolling();
  },
  watch: {
    searchTerm() {
      this.output = [];
      this.after = undefined;
      this.loading = true;
      this.error = false;
      this.getData(this.searchTerm, this.after);
    },
  },
};
</script>

<style lang="scss" scoped>
.home {
  width: 100%;
  position: relative;
}

.title {
  display: flex;
  justify-content: space-between;
  align-items: center;

  .score {
    background: #42b983;
    color: white;
    border-radius: 5px;
    padding: 3px 6px;
  }
}

.form {
  width: 60%;
  margin: 0 auto 40px auto;
  height: 35px;
  display: flex;
  justify-content: space-between;

  input {
    width: 80%;
    padding: 8px;
    font-size: 18px;
    border: none;
    outline: none;
    box-shadow: inset 0px 0px 5px 0px rgba(0, 0, 0, 0.3);
  }

  button {
    width: 20%;
    border: none;
    outline: none;
    background: #42b983;
    color: white;
    cursor: pointer;
  }
}

.results {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  width: 100%;
  // min-height: 1000px;
  font-size: 8px;

  h3 {
    font-size: 2rem;
    color: pink;
    margin-top: 100px;
  }

  .card {
    width: 32%;
    margin: 10px 5px;
    background: white;
    height: fit-content;
    box-shadow: 0 0 8px 0 rgba(0, 0, 0, 0.3);
    border-radius: 3px;
    overflow: hidden;

    img {
      width: 100%;
      height: auto;
      display: inline-block;
      box-shadow: 0 0 8px 0 rgba(0, 0, 0, 0.2);
    }

    .title {
      color: gray;
      font-size: 1.2rem;
      padding: 12px 24px;
    }

    .footer {
      display: flex;
      justify-content: space-between;
      align-items: center;

      .date-author {
        padding: 8px 24px;
        font-size: 0.75rem;

        .author {
          color: #42b983;
          text-decoration: none;
          font-weight: bold;
        }
      }

      .heart {
        margin-right: 38px;
        position: relative;
        cursor: pointer;

        &::after,
        &::before {
          content: '';
          position: absolute;
          width: 10px;
          height: 15px;
          background: #42b983;
          transform-origin: center;
          border-top-right-radius: 1000px;
          border-top-left-radius: 1000px;
          transform: translateY(-50%);
        }

        &::after {
          transform: translateY(-50%) rotate(45deg);
        }
        &::before {
          transform: translateY(-50%) translateX(-50%) rotate(-45deg);
        }
      }
    }
  }
}

.loading-image {
  width: 400px;
  height: 300px;
}

.scroll-up {
  padding: 5px 10px 8px 10px;
  border-radius: 3px;
  position: fixed;
  right: 5px;
  bottom: 30px;
  font-size: 2rem;
  color: white;
  background: #42b983;
  border: none;
  outline: none;
  cursor: pointer;
  opacity: 1;
  animation: animate 1s ease-in-out infinite alternate, fadein 1s ease-in-out;
}

@keyframes fadein {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes animate {
  from {
    transform: translateY(0);
  }
  to {
    transform: translateY(20px);
  }
}

@media screen and (max-width: 950px) {
  .results {
    .card {
      width: 47%;
    }
  }
}

@media screen and (max-width: 500px) {
  .results {
    .card {
      width: 90%;
    }
  }
}

@media screen and (max-width: 450px) {
  .form {
    width: 80%;

    input {
      width: 70%;
    }

    button {
      width: 30%;
    }
  }

  .scroll-up {
    padding: 2px 4px 4px 4px;
  }
}
</style>
