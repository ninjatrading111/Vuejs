<template>
  <div class="calendar-container">
    <div class="scroll-buttons">
      <button @click="prevPage" class="nav-button">&lt;</button>
      <div class="dates-container" ref="datesContainer" @scroll="updateScrollbarWidth">
        <div 
          v-for="(date, index) in dates" 
          :key="index" 
          :class="['date-item', { active: index === activeIndex }]" 
          @click="setActive(index)"
        >
          {{ formatDate(date) }}
        </div>
      </div>
      <button @click="nextPage" class="nav-button">&gt;</button>
    </div>
    <div class="selected-date">
      <p>{{ selectedDate }}</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      dates: [],
      activeIndex: 0,
    };
  },
  computed: {
    selectedDate() {
      return this.formatDate(this.dates[this.activeIndex]);
    },
  },
  methods: {
    setActive(index) {
      this.activeIndex = index;
      this.scrollToActiveDate();
    },
    prevPage() {
      if (this.activeIndex > 0) {
        this.activeIndex--;
        this.scrollToActiveDate();
      }
    },
    nextPage() {
      if (this.activeIndex < this.dates.length - 1) {
        this.activeIndex++;
        this.scrollToActiveDate();
      }
    },
    scrollToActiveDate() {
      const container = this.$refs.datesContainer;
      const activeElement = container.children[this.activeIndex];
      container.scrollLeft = activeElement.offsetLeft - container.offsetLeft - container.clientWidth / 2 + activeElement.clientWidth / 2;
      this.updateScrollbarWidth();
    },
    updateScrollbarWidth() {
      const container = this.$refs.datesContainer;
      const scrollWidth = container.scrollWidth;
      const clientWidth = container.clientWidth;
      const scrollLeft = container.scrollLeft;
      const maxScroll = scrollWidth - clientWidth;

      // Calculate the ratio of scrolled distance to total scrollable distance
      const scrollRatio = scrollLeft / maxScroll;

      // Calculate new length of scrollbar
      const minLength = 10; // Minimum length for the scrollbar
      const maxLength = 50; // Maximum length for the scrollbar
      const newLength = maxLength - (maxLength - minLength) * scrollRatio;

      // Set the width of the scrollbar
      container.style.setProperty('--scrollbar-length', `${newLength}%`);
    },
    formatDate(dateString) {
      const date = new Date(dateString);
      const month = date.toLocaleString('en-us', { month: 'short' });
      const day = date.getDate();
      return `${month} ${day}`;
    },
    generateDates() {
      const today = new Date(); // Today's date
      const startDate = new Date(today.getFullYear(), today.getMonth(), today.getDate()); // Start date from today

      const endDate = new Date(today.getFullYear(), 11, 31); // December 31 of the current year

      const dates = [];
      let currentDate = new Date(startDate);

      while (currentDate <= endDate) {
        dates.push(currentDate.toLocaleDateString('en-US', { month: 'short', day: 'numeric' })); // Push date in "MMM D" format
        currentDate.setDate(currentDate.getDate() + 1); // Move to next day
      }

      this.dates = dates;
    },
  },
  mounted() {
    this.generateDates(); // Generate dates when component is mounted
    this.$nextTick(() => {
      this.scrollToActiveDate(); // Scroll to active date after initial render
    });
  },
};
</script>

<style scoped>
.calendar-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 80%;
  margin: 0 auto;
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 10px;
  background-color: #f5f5f5;
  overflow: hidden; /* Ensure overflow does not leak out of container */
}

.scroll-buttons {
  display: flex;
  justify-content: space-between;
  width: 100%;
}
.nav-button {
  background-color: #ddd;
  border: none;
  padding: 10px;
  cursor: pointer;
  margin: 5px;
  border-radius: 4px;
}
.dates-container {
  display: flex;
  overflow-x: auto;
  width: 100%; /* Ensure the container takes up the full width */
  padding: 10px 0;
}

/* WebKit scrollbar styles */
.dates-container::-webkit-scrollbar {
  width: 8px; /* Adjust width if needed */
}

.dates-container::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 4px;
  width: var(--scrollbar-length); /* Dynamically adjust height based on scroll ratio */
}

.date-item {
  padding: 10px;
  margin: 0 5px;
  cursor: pointer;
  white-space: nowrap;
  border-radius: 4px;
  transition: background-color 0.3s;
}
.date-item.active {
  background-color: #ccc;
}
.selected-date {
  width: 100%;
  text-align: center;
  margin-top: 20px;
  font-size: 18px;
  font-weight: bold;
  background-color: #d4d4d4;
  height: 200px;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
