<template>
  <q-page class="relative-position">
   <q-scroll-area class="absolute full-width full-height">
   <div class="q-py-lg q-px-md row items-end q-col-gutter-md">
      <div class="col">
        <q-input
          v-model="newTweetContent"
          class="new-Tweet"
          placeholder="What's happening?"
          maxlength="280"
          bottom-slots
          counter
          autogrow
        >
          <template v-slot:before>
            <q-avatar size="xl">
              <img src="https://images.mubicdn.net/images/cast_member/3071/cache-3195-1568084972/image-w856.jpg?size=800x">
            </q-avatar>
          </template>
        </q-input>
      </div>
      <div class="col col-shrink">
        <q-btn
          @click="addNewTweet"
          :disable="!newTweetContent"
          class="q-mb-lg"
          color="primary"
          label="Tweet"
          rounded
          unelevated
          no-caps
        />
      </div>
    </div>
    <q-separator
      class="divider"
      color="grey-2"
      size="10px"
    />
    <q-list separator>
        <transition-group
          appear
          enter-active-class="animated fadeIn slow"
          leave-active-class="animated fadeOut slow"
        >
          <q-item
            v-for="tweet in tweets"
            :key="tweet.id"
            class="tweet q-py-md"
          >
            <q-item-section avatar top>
              <q-avatar size="xl">
                <img src="https://images.mubicdn.net/images/cast_member/3071/cache-3195-1568084972/image-w856.jpg?size=800x">
              </q-avatar>
            </q-item-section>

            <q-item-section>
              <q-item-label class="text-subtitle1">
                <strong>Danny Connell</strong>
                <span class="text-grey-7">
                  @danny__connell
                  <br class="lt-md">&bull; {{ relativeDate(tweet.date) }}
                </span>
              </q-item-label>
              <q-item-label class="tweet-content text-body1">{{ tweet.content }}</q-item-label>
              <div class="tweet-icons row justify-between q-mt-sm">
                <q-btn
                  color="grey"
                  icon="far fa-comment"
                  size="sm"
                  flat
                  round
                />
                <q-btn
                  color="grey"
                  icon="fas fa-retweet"
                  size="sm"
                  flat
                  round
                />
                <q-btn
                  @click="toggleLiked(tweet)"
                  :color="tweet.liked ? 'pink' : 'grey'"
                  :icon="tweet.liked ? 'fas fa-heart' : 'far fa-heart'"
                  size="sm"
                  flat
                  round
                />
                <q-btn
                  @click="deletetweet(tweet)"
                  color="grey"
                  icon="fas fa-trash"
                  size="sm"
                  flat
                  round
                />
              </div>
            </q-item-section>
          </q-item>
        </transition-group>
      </q-list>
    </q-scroll-area>
  </q-page>
</template>

<script>
import { defineComponent } from 'vue'
import { formatDistance } from 'date-fns'
import db from 'src/boot/firebase'
import { collection, query, onSnapshot, orderBy, addDoc, doc, deleteDoc, updateDoc } from 'firebase/firestore'

export default defineComponent({
  name: 'PageHome',
  data () {
    return {
      newTweetContent: '',
      tweets: [],
      unsubscribe: null
    }
  },
  methods: {
    relativeDate (value) {
      return formatDistance(value, new Date())
    },
    async addNewTweet () {
      const newTweet = {
        id: '1',
        content: this.newTweetContent,
        date: Date.now(),
        liked: false
      }
      await addDoc(collection(db, 'tweets'), newTweet)
      this.newTweetContent = ''
    },
    async deletetweet (tweet) {
      await deleteDoc(doc(db, 'tweets', tweet.id))
    },
    async toggleLiked (tweet) {
      const washingtonRef = doc(db, 'tweets', tweet.id)
      await updateDoc(washingtonRef, {
        liked: !tweet.liked
      })
    }
  },
  mounted () {
    const q = query(collection(db, 'tweets'), orderBy('date'))
    this.unsubscribe = onSnapshot(q, (snapshot) => {
      snapshot.docChanges().forEach((change) => {
        const tweetChange = change.doc.data()
        tweetChange.id = change.doc.id
        if (change.type === 'added') {
          this.tweets.unshift(tweetChange)
        }
        if (change.type === 'modified') {
          const index = this.tweets.findIndex(tweet => tweet.id === tweetChange.id)
          Object.assign(this.tweets[index], tweetChange)
        }
        if (change.type === 'removed') {
          const index = this.tweets.findIndex(tweet => tweet.id === tweetChange.id)
          this.tweets.splice(index, 1)
        }
      })
    })
  },
  beforeUnmount () {
    this.unsubscribe()
  }
})
</script>
