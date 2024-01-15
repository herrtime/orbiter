<template>
  <div
    class="orbiter"
    :class="{ 'orbiter--loading': loading }"
  >
    <Orbit
      v-for="(orbitDate, orbitIndex) in dates.slice(0, 9)"
      :key="orbitDate"
      :date="orbitDate.contact_date ? getDateStr(orbitDate.contact_date) : ''"
      :number="9 - orbitIndex"
      :items="orbitDate.array"
      :animate="animate"
    />
    <a
      href="javascript:{}"
      class="orbiter__menu"
    >
      <svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M13.6498 2.35C12.1998 0.9 10.2098 0 7.99977 0C3.57977 0 0.00976562 3.58 0.00976562 8C0.00976562 12.42 3.57977 16 7.99977 16C11.7298 16 14.8398 13.45 15.7298 10H13.6498C12.8298 12.33 10.6098 14 7.99977 14C4.68977 14 1.99977 11.31 1.99977 8C1.99977 4.69 4.68977 2 7.99977 2C9.65977 2 11.1398 2.69 12.2198 3.78L8.99977 7H15.9998V0L13.6498 2.35Z" fill="white"/>
      </svg>
    </a>
  </div>
</template>

<script>
import Orbit from '@/components/Orbit.vue';
import moment from 'moment';

export default {
  components: {
    Orbit
  },
  data() {
    return {
      dates: [],
      currentDate: null,
      animate: false, // false|'top'|'bottom'
      loading: false, // loading progress
    }
  },
  watch: {
    currentDate() {
      this.getItems(); // load new items when date is changed
    },
  },
  mounted() {
    // set today as default date
    this.currentDate = moment().format('YYYY-MM-DD');
    // add event for navigate
    document.documentElement.addEventListener('keyup', (evt) => {
      // prevent change date when loading in progress
      if (this.loading) return false;
      const d = moment(this.currentDate);
      switch (evt.key) {
        case 'ArrowUp':
          if (d.diff(moment(), 'days') < 0) {
            this.currentDate = d.add(1, 'days').format('YYYY-MM-DD');
            this.animate = 'top';
          }
          break;
        case 'ArrowDown':
          this.currentDate = d.add(-1, 'days').format('YYYY-MM-DD');
          this.animate = 'bottom';
          break;
      }
    });
  },
  methods: {
    getDateStr(date) {
      // return a formatted date
      const d = moment(date);
      if (d.diff(moment(), 'days') === 0) {
        return 'Today';
      }
      return d.format('ddd MMM D');
    },
    getItems() {
      // items loading
      if (this.loading) return;
      this.loading = true;
      const d = moment(this.currentDate);
      if (d.diff(moment(), 'days')) {
        d.add(1, 'days');
      }
      fetch(`https://xwmi-5dlx-wkyj.t7.xano.io/api:oUvfVMO5/receive_week?start_date=${d.format('YYYY-M-D')}`)
        .then(response => response.json())
        .then((response) => {
          this.prepareData(response || []);
        })
        .catch((error) => {
          console.error(error);
        })
        .finally(() => {
          this.loading = false;
        })
    },
    prepareData(items) {
      // prepare items to view
      if (!this.dates.length) {
        // if dates is empty - just add loaded items (current date is today)
        // and add first empty item (we always need previous and next dates for animation)
        this.dates = items;
        this.dates.unshift({});
        return;
      }
      // do the magic - remove one element and add another from loaded items
      if (this.animate === 'top') {
        this.dates.pop();
        const last = items.shift();
        this.dates.unshift(last);
      } else if (this.animate === 'bottom') {
        this.dates.shift();
        const first = items.pop();
        console.log('first', first.contact_date);
        this.dates.push(first);
      }
    },
  }
}
</script>

<style lang="sass" scoped>
.orbiter
  width: 100vw
  height: 100vh
  overflow: hidden

  &--loading
   pointer-events: none

  &__menu
    display: flex
    justify-content: center
    align-items: center
    border-radius: 50%
    border: 1px solid #fff
    width: 75px
    height: 75px
    background: #000 url(/src/assets/company.png) no-repeat 50% 50%
    box-shadow: 0 4px 82px 0 #000000
    position: absolute
    left: 50%
    bottom: 30px
    transform: translateX(-50%)

    .orbiter--loading &
      background-image: none

      svg
        display: block
        animation: loading 1s infinite

    svg
      display: none

@keyframes loading
  from
    transform: rotate(0deg)
  to
    transform: rotate(360deg)
</style>
