<template>
  <div
    class="orbit"
    :class="{
      'orbit--next': number === 1,
      'orbit--current': number === 8,
      'orbit--prev': number === 9,
    }"
    :style="getOrbitStyles()"
  >
    <div
      class="orbit__item"
      :style="getOrbitItemStyles()"
    />
    <div
      class="orbit__date"
      v-text="date"
    />
    <div
      class="orbit__person"
      v-for="(person, personIndex) in visiblePersons"
      :style="getPersonPosition(personIndex)"
      @mouseenter="currentPerson = person.id"
      @mouseleave="currentPerson = null"
    >
      <div class="orbit__person-photo">
        <img
          :src="person.img"
          :alt="person.name"
        />
      </div>
      <div class="orbit__person-card">
        <div class="orbit__person-outer">
          <div class="orbit__person-inner">
            <div class="orbit__person-name" v-text="person.name" />
            <div class="orbit__person-position" v-text="person.position" />
            <div class="orbit__person-city" v-text="person.city" />
            <div class="orbit__person-mutual">
              <img src="https://i.pravatar.cc/150?img=1" alt="" />
              <img src="https://i.pravatar.cc/150?img=2" alt="" />
              <img src="https://i.pravatar.cc/150?img=3" alt="" />
              <img src="https://i.pravatar.cc/150?img=4" alt="" />
              <span>Jason Diamond, John Eremic, and 281 other mutual connections</span>
            </div>
          </div>
          <div class="orbit__person-company">
            <img src="../assets/company.png" alt="">
          </div>
        </div>
        <div
          v-if="currentPerson === person.id && messagesShow"
          class="orbit__messages"
        >
          <div class="orbit__message">
            <div class="orbit__message-button">
              <svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M3 7.75C3 7.21957 3.21071 6.71086 3.58579 6.33579C3.96086 5.96071 4.46957 5.75 5 5.75H19C19.5304 5.75 20.0391 5.96071 20.4142 6.33579C20.7893 6.71086 21 7.21957 21 7.75M3 7.75V17.75C3 18.2804 3.21071 18.7891 3.58579 19.1642C3.96086 19.5393 4.46957 19.75 5 19.75H19C19.5304 19.75 20.0391 19.5393 20.4142 19.1642C20.7893 18.7891 21 18.2804 21 17.75V7.75M3 7.75L12 13.75L21 7.75" stroke="white" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
              </svg>
              <span>Reply from Emery Wells</span>
            </div>
            <div class="orbit__message-inner">
              <div class="orbit__message-date">
                <span>Saturday, November 4 2023 at 9:04 AM EST</span>
                <span>2 days ago</span>
              </div>
              <div class="orbit__message-subject" v-text="person._orbits_last_message.message_head" />
              <div
                class="orbit__message-content"
                v-text="person._orbits_last_message.message"
              />
              <div
                class="orbit__message-toggle"
                data-more="More"
                data-less="Less"
                @click="toggleMessage"
              >
                <svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                  <path d="M6 9.75L12 15.75L18 9.75" stroke="white" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    number: { // orbit number include previous and next. begin from bottom
      type: Number,
      required: true
    },
    date: { // date to show on orbit
      type: Date,
      required: true
    },
    items: { // list of the people
      type: Array,
      default: [],
    },
    animate: { // direction (or false) of animate
      type: Boolean, String,
      default: false,
    },
  },
  computed: {
    radius() {
      // Get radius of the orbit (based on window height)
      return window.innerHeight / 9 * this.number;
    },
    visiblePersons() {
      // Get only visible persons on orbit
      // Here 80 is a size of each person
      // todo: But here is the first problem - people can appear on bigger orbit. So now this is commented
      // const max = Math.PI * this.radius / 80;
      // return this.items.slice(0, max);
      return this.items;
    },
  },
  data() {
    return {
      currentPerson: null, // opened person (we don't need to show all messages for all persons)
      messageTimeout: null, // show messages with timeout
      messagesShow: false, // show or hide messages
    }
  },
  watch: {
    currentPerson (val) {
      // timeout for show messages only for current people (we don't need to draw all messages for all people)
      if (val) {
        clearTimeout(this.messageTimeout);
        this.messageTimeout = setTimeout(() => {
          this.messagesShow = true;
        }, 600); // 300 for animate card + 300 timeout for message showing
      } else {
        clearTimeout(this.messageTimeout);
        this.messagesShow = false;
      }
    },
  },
  methods: {
    getOrbitStyles() {
      // Build styles for orbit
      const styles = [];
      styles.push(`margin-left:${-this.radius}px;`);
      styles.push(`width:${this.radius * 2}px;`);
      styles.push(`height:${this.radius}px;`);
      styles.push(`border-top-left-radius:${this.radius}px;`);
      styles.push(`border-top-right-radius:${this.radius}px;`);
      return styles.join('');
    },
    getOrbitItemStyles() {
      // Build styles for orbit
      const styles = [];
      styles.push(`border-top-left-radius:${this.radius}px;`);
      styles.push(`border-top-right-radius:${this.radius}px;`);
      return styles.join('');
    },
    getPersonPosition(index) {
      // Build styles (position) for person
      const n = this.visiblePersons.length + 1;
      const start = Math.floor(-n / 2) + (n % 2 === 0 ? 1 : 1.5);
      const styles = [];
      let offset = 0;
      if (this.number === 8 && this.visiblePersons.length % 2 && index === Math.floor(this.visiblePersons.length / 2)) {
        // for center element (if exists) on first visible orbit add correction - we need to show date
        offset = 100 / this.radius
      }
      styles.push(`top:${String(this.radius + -this.radius * Math.cos((1 / n) * (index + start) * Math.PI - offset))}px;`);
      styles.push(`left:${String(this.radius + this.radius * Math.sin((1 / n + offset) * (index + start) * Math.PI - offset))}px;`);
      return styles.join('');
    },
    toggleMessage(evt) {
      // Toggle more or less message with vanilla javascript - this is faster
      // todo: Here is the second problem - when click 'less' card can be hidden because mouse leave it (the height of the card changes)
      const message = evt.target.closest('.orbit__message');
      const content = message.querySelector('.orbit__message-content');
      if (message.getAttribute('aria-expanded') === 'true') {
        message.setAttribute('aria-expanded', 'false');
        const startHeight = content.offsetHeight;
        content.style.whiteSpace = 'nowrap';
        const endHeight = content.offsetHeight;
        content.style.whiteSpace = 'initial';
        content.style.maxHeight = `${startHeight}px`;
        setTimeout(() => {
          content.style.maxHeight = `${endHeight}px`;
        }, 10);
        content.addEventListener('transitionend', () => {
          content.style.whiteSpace = 'nowrap';
          content.style.maxHeight = '';
        }, { once: true });
      } else {
        message.setAttribute('aria-expanded', 'true');
        const startHeight = content.offsetHeight;
        content.style.whiteSpace = 'initial';
        const endHeight = content.offsetHeight;
        content.style.maxHeight = `${startHeight}px`;
        setTimeout(() => {
          content.style.maxHeight = `${endHeight}px`;
        }, 10);
        content.addEventListener('transitionend', () => {
          content.style.maxHeight = '';
        }, { once: true });
      }
    }
  }
}
</script>

<style lang="sass">
$animationSpeed: .3s

.orbit
  position: absolute
  bottom: -1px
  left: 50%
  visibility: visible
  opacity: 1
  transition: margin $animationSpeed, width $animationSpeed, height $animationSpeed, visibility 0s, opacity $animationSpeed

  &--next
    visibility: hidden
    opacity: 0
    transition-delay: 0s, 0s, 0s, $animationSpeed, 0s

  &--prev
    visibility: hidden
    opacity: 0
    transition-delay: 0s, 0s, 0s, $animationSpeed, 0s

  &__item
    display: block
    border-radius: 0
    border: 1px solid #fff
    width: 100%
    height: 100%
    transform: scale(1)
    transform-origin: 50% 100%
    transition: all $animationSpeed
    pointer-events: none

    &::before
      content: ''
      display: block
      border-radius: inherit
      width: calc(100% + 10px)
      height: calc(100% + 10px)
      background: linear-gradient(0deg, #000, #0000 100%) border-box
      position: absolute
      top: -5px
      left: -5px

  &__date
    padding: 7px
    font-weight: 300
    color: #929292
    background: #0A0A0A
    position: absolute
    top: 0
    left: 50%
    visibility: hidden
    opacity: 0
    transition: visibility 0s $animationSpeed, opacity $animationSpeed 0s
    transform: translate(-50%, -50%)

    .orbit--current &
      visibility: visible
      opacity: 1
      transition-delay: 0s

  &__person
    font-size: 12px
    line-height: 1.25em
    color: #ffffff
    position: absolute
    z-index: 1
    transition: all $animationSpeed

    &:hover,
    &--current
      z-index: 2

    &-photo
      display: block
      margin: -30px 0 0 -30px
      border-radius: 50%
      border: 1px solid transparent
      background: linear-gradient(0deg, #000, #fff) border-box
      width: 60px
      height: 60px
      overflow: hidden
      transition: transform 0s
      position: relative
      z-index: 4

      .orbit__person:hover &,
      .orbit__person.orbit--current &
        transition-duration: $animationSpeed
        transform: scale(1.66)

      img
        display: block
        width: 100%
        height: 100%
        object-fit: cover

    &-card
      width: 502px
      min-height: 122px
      position: absolute
      top: -60px
      left: -60px
      z-index: 1
      visibility: hidden
      opacity: 0
      transition: visibility 0s 0s, opacity 0s 0s
      pointer-events: none

      border-radius: 10px
      border: 1px solid transparent
      background: linear-gradient(0deg, #2c2c2c, #b5b5b5) border-box

      .orbit__person:hover &,
      .orbit__person.orbit--current &
        visibility: visible
        opacity: 1
        transition-duration: 0s, $animationSpeed
        transition-delay: $animationSpeed
        pointer-events: initial

    &-outer
      display: flex
      border-radius: 10px
      padding: 9px 10px 9px 125px
      min-height: 118px
      background: #0a0a0a

      &:not(:last-child)
        border-radius: 10px 10px 0 0

    &-inner
      display: flex
      flex-direction: column

    &-name,
    &-position,
    &-city
      margin-bottom: 10px
      font-size: 12px
      line-height: 15px

    &-name
      font-weight: bold
      font-size: 16px
      line-height: 19px

    &-mutual
      display: flex
      align-items: center
      margin-top: auto

      img
        display: block
        border: 1px solid #929292
        border-radius: 50%
        width: 20px
        height: 20px
        object-fit: cover
        object-position: 50% 50%

        + img
          margin-left: -4px

      span
        display: block
        margin-left: 5px

    &-company
      flex-shrink: 0
      margin-left: 10px
      width: 50px

      img
        border: 1px solid transparent
        background: linear-gradient(0deg, #2c2c2c, #b5b5b5) border-box
        border-radius: 4px
        display: block
        width: 50px
        height: 50px

  &__messages
    border-radius: 0 0 10px 10px
    padding: 10px
    background: linear-gradient(0deg, rgba(10, 10, 10, 0.65) 0%, rgba(10, 10, 10, 0.65) 100%), linear-gradient(180deg, rgba(0, 0, 0, 0.20) 0%, rgba(0, 0, 0, 0.00) 100%), rgba(255, 255, 255, 0.05)

  &__message
    border-radius: 10px

    &-button
      display: flex
      align-items: center
      border-radius: 10px 10px 0 0
      padding: 10px
      min-height: 44px
      font-size: 16px
      line-height: 24px
      background: rgba(#0a0a0a, .85)
      cursor: pointer

      svg
        display: block
        width: 24px
        height: 24px
        margin-right: 10px

    &-inner
      padding: 10px
      background: rgba(#0a0a0a, .65)
      
    &-date
      display: flex
      justify-content: space-between
      margin-bottom: 10px
      color: #929292

    &-subject
      margin-bottom: 5px
      font-size: 16px
      line-height: 19px
      font-weight: 700

    &-content
      font-size: 16px
      line-height: 19px
      overflow: hidden
      white-space: nowrap
      text-overflow: ellipsis
      transition: max-height $animationSpeed

    &-toggle
      margin-top: 5px
      display: flex
      align-items: center
      cursor: pointer

      &::before
        content: attr(data-more)

        .orbit__message[aria-expanded="true"] &
          content: attr(data-less)

      svg
        transform: rotate(0)
        transition: transform $animationSpeed

        .orbit__message[aria-expanded="true"] &
          transform: rotate(180deg)

</style>
