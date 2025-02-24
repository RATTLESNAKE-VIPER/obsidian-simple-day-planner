<script lang="ts">
  import { onMount } from "svelte";
  import { onDestroy } from "svelte";
  import { planSummary, now, nowPosition, zoomLevel } from "./timeline-store";
  import { TIMELINE_DEFAULT_ZOOM } from "./constants";
  import type { PlanItem, PlanSummaryData } from "./plan-data";
  const moment = (window as any).moment;

  export let summary: PlanSummaryData;
  export let rootEl: HTMLElement;
  let timelineZoomLevel: number;
  let position: number;
  let timelineMeterPosition: number;
  let currentTime: Date;
  let autoScroll: boolean = true;

  const unsubSummary = planSummary.subscribe((val) => {
    summary = val;
  });

  const unsubPosition = nowPosition.subscribe((val) => {
    position = val;
  });

  const unsubCurrentTime = now.subscribe((val) => {
    currentTime = val;
    scrollToPosition(position - 150);
    if (!timelineZoomLevel) {
      timelineZoomLevel = TIMELINE_DEFAULT_ZOOM;
    }
  });

  const unsubSettings = zoomLevel.subscribe((val) => {
    timelineZoomLevel = val;
  });

  onDestroy(unsubSummary);
  onDestroy(unsubPosition);
  onDestroy(unsubCurrentTime);
  onDestroy(unsubSettings);
  onDestroy(removeScrollListener);

  onMount(() => {
    addScrollListener();
  });

  function addScrollListener() {
    rootEl.addEventListener("scroll", disableAutoScroll);
  }

  function removeScrollListener() {
    rootEl.removeEventListener("scroll", disableAutoScroll);
  }

  function disableAutoScroll(ev: any) {
    autoScroll = false;
  }

  function scrollToPosition(position: number) {
    if (autoScroll && !summary.current?.isEnd) {
      removeScrollListener();
      rootEl.scrollTo({ left: 0, top: position, behavior: "smooth" });
      setTimeout(addScrollListener, 1000);
    }
  }

  function shortClass(item: PlanItem) {
    return item.durationMins < 75 / timelineZoomLevel ? "short" : "";
  }

  function pastClass(item: PlanItem) {
    return item.isPast ? "past" : "";
  }
</script>

{#if summary.items.length > 0}
  <div id="scroll-controls">
    <label for="auto-scroll">Track current time</label>
    <input
      id="auto-scroll"
      type="checkbox"
      class="toggle"
      bind:checked={autoScroll}
    />
  </div>
  <div id="day-planner-timeline-container">
    <div
      class="aside aside-x{timelineZoomLevel} filled"
    >
      <div class="aside__line filled__line">
        <div class="filled__line__completed" style="height: {nowPosition}px;" />
      </div>
    </div>

    <div class="events">
      {#each summary.items as item, i}
        <div
          class="event_item event_item_color{(i % 10) + 1} {shortClass(
            item
          )} {pastClass(item)}"
          style="height: {item.durationMins * timelineZoomLevel}px;"
          data-title={item.rawTime}
        >
          <div class="event_item_contents">
            <div
              class="ei_Dot {item === summary.current ? 'dot_active' : ''}"
            />
            <div class="ei_Title">{item.rawTime}</div>
            <div class="ei_Copy">{item.text ?? ""}</div>
          </div>
        </div>
      {/each}
    </div>

    <div
      id="now-line"
      style="top:{position}px; display: {summary.current &&
      !summary.current.isEnd
        ? 'block'
        : 'none'}"
    >
      <span class="timeline-time">{moment(currentTime).format("HH:mm")}</span>
    </div>
  </div>
{:else}
  <div class="empty-timeline">No Planner Data</div>
{/if}

<style>
  #day-planner-timeline-container {
    position: relative;
    border-radius: var(--size-4-3);
    overflow: hidden;
    margin-bottom: var(--size-4-9);

    --skobeloff: rgba(0, 100, 102, 0.5);
    --midnight-green-eagle-green: rgba(6, 90, 96, 0.5);
    --midnight-green-eagle-green-2: rgba(11, 82, 91, 0.5);
    --midnight-green-eagle-green-3: rgba(20, 69, 82, 0.5);
    --charcoal: rgba(27, 58, 75, 0.5);
    --prussian-blue: rgba(33, 47, 69, 0.5);
    --space-cadet: rgba(39, 38, 64, 0.5);
    --dark-purple: rgba(49, 34, 68, 0.5);
    --russian-violet: rgba(62, 31, 71, 0.5);
    --russian-violet-2: rgba(77, 25, 77, 0.5);
  }

  .aside {
    position: absolute;
    top: -1px;
    left: 0;
    width: 65px;
    height: 100%;

    background-repeat: repeat-y;
    opacity: 80%;
  }

  .aside-x5 {
    background-image: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABYAAAEsCAYAAADHIkNEAAAAAXNSR0IArs4c6QAAAFBlWElmTU0AKgAAAAgAAgESAAMAAAABAAEAAIdpAAQAAAABAAAAJgAAAAAAA6ABAAMAAAABAAEAAKACAAQAAAABAAAAFqADAAQAAAABAAABLAAAAAAuMW7GAAABWWlUWHRYTUw6Y29tLmFkb2JlLnhtcAAAAAAAPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyIgeDp4bXB0az0iWE1QIENvcmUgNS40LjAiPgogICA8cmRmOlJERiB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPgogICAgICA8cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0iIgogICAgICAgICAgICB4bWxuczp0aWZmPSJodHRwOi8vbnMuYWRvYmUuY29tL3RpZmYvMS4wLyI+CiAgICAgICAgIDx0aWZmOk9yaWVudGF0aW9uPjE8L3RpZmY6T3JpZW50YXRpb24+CiAgICAgIDwvcmRmOkRlc2NyaXB0aW9uPgogICA8L3JkZjpSREY+CjwveDp4bXBtZXRhPgpMwidZAAAAsElEQVR4Ae3ZMQoAIAgFUL3/ocsO0GBLFE9wESJ8jj8jYlQrAgQIECBAgAABAgQIECBAgAABAgQIECBAgACBbwWyNutGQuuNIkCAAAECBAgQIECAAAECBAgQIECAAAECBAgQeEbgJBLaLScq2smYEyBAgAABAgQIECBAgAABAgQIECBAgAABAgSuCpxEQqKfqyfzOQECBAgQIECAAAECBAgQIECAAAECBAgQIECgKzABwWUEBHBCyqYAAAAASUVORK5CYII=);
  }

  .aside-x4 {
    background-image: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABYAAADwCAYAAAAEqZSkAAAABGdBTUEAALGPC/xhBQAAACBjSFJNAAB6JgAAgIQAAPoAAACA6AAAdTAAAOpgAAA6mAAAF3CculE8AAAAeGVYSWZNTQAqAAAACAAEARIAAwAAAAEAAQAAARoABQAAAAEAAAA+ARsABQAAAAEAAABGh2kABAAAAAEAAABOAAAAAAAAAEgAAAABAAAASAAAAAEAA6ABAAMAAAABAAEAAKACAAQAAAABAAAAFqADAAQAAAABAAAA8AAAAABUK6+SAAAACXBIWXMAAAsTAAALEwEAmpwYAAACMWlUWHRYTUw6Y29tLmFkb2JlLnhtcAAAAAAAPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyIgeDp4bXB0az0iWE1QIENvcmUgNS40LjAiPgogICA8cmRmOlJERiB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPgogICAgICA8cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0iIgogICAgICAgICAgICB4bWxuczp0aWZmPSJodHRwOi8vbnMuYWRvYmUuY29tL3RpZmYvMS4wLyIKICAgICAgICAgICAgeG1sbnM6ZXhpZj0iaHR0cDovL25zLmFkb2JlLmNvbS9leGlmLzEuMC8iPgogICAgICAgICA8dGlmZjpPcmllbnRhdGlvbj4xPC90aWZmOk9yaWVudGF0aW9uPgogICAgICAgICA8ZXhpZjpDb2xvclNwYWNlPjE8L2V4aWY6Q29sb3JTcGFjZT4KICAgICAgICAgPGV4aWY6UGl4ZWxYRGltZW5zaW9uPjIyPC9leGlmOlBpeGVsWERpbWVuc2lvbj4KICAgICAgICAgPGV4aWY6UGl4ZWxZRGltZW5zaW9uPjMwMDwvZXhpZjpQaXhlbFlEaW1lbnNpb24+CiAgICAgIDwvcmRmOkRlc2NyaXB0aW9uPgogICA8L3JkZjpSREY+CjwveDp4bXBtZXRhPgrL2/xqAAAA2klEQVR4Ae3bMQqEMBBA0QT2Bh7G+5d2e5G9wRZruliM6CCBsE8QQghxeOUHaynlvb/zPHUf9TXPuCYlQIAAAQIECBAgQIAAAQIECBAg8N8CLegtNwk+V863i7crB7sza7cev8y0ze/4MX2RAAECBAgQIECAAAECBAgQIECAAIEmkGmbkdyheWbaZnTxmOaZaZvRxJpnJGOfAAECBAgQIECAAAECBAgQIECAwEMCmbZ5aJjRHJm2OaZhnk189590DTPStE+AAAECBAgQIECAAAECBAgQIEBgVoEfgLcJ4JWKwMAAAAAASUVORK5CYII=);
  }

  .aside-x3 {
    background-image: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABYAAAC0CAYAAACQYNzeAAAABGdBTUEAALGPC/xhBQAAACBjSFJNAAB6JgAAgIQAAPoAAACA6AAAdTAAAOpgAAA6mAAAF3CculE8AAAAeGVYSWZNTQAqAAAACAAEARIAAwAAAAEAAQAAARoABQAAAAEAAAA+ARsABQAAAAEAAABGh2kABAAAAAEAAABOAAAAAAAAAEgAAAABAAAASAAAAAEAA6ABAAMAAAABAAEAAKACAAQAAAABAAAAFqADAAQAAAABAAAAtAAAAAD5WFEbAAAACXBIWXMAAAsTAAALEwEAmpwYAAACMWlUWHRYTUw6Y29tLmFkb2JlLnhtcAAAAAAAPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyIgeDp4bXB0az0iWE1QIENvcmUgNS40LjAiPgogICA8cmRmOlJERiB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPgogICAgICA8cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0iIgogICAgICAgICAgICB4bWxuczp0aWZmPSJodHRwOi8vbnMuYWRvYmUuY29tL3RpZmYvMS4wLyIKICAgICAgICAgICAgeG1sbnM6ZXhpZj0iaHR0cDovL25zLmFkb2JlLmNvbS9leGlmLzEuMC8iPgogICAgICAgICA8dGlmZjpPcmllbnRhdGlvbj4xPC90aWZmOk9yaWVudGF0aW9uPgogICAgICAgICA8ZXhpZjpDb2xvclNwYWNlPjE8L2V4aWY6Q29sb3JTcGFjZT4KICAgICAgICAgPGV4aWY6UGl4ZWxYRGltZW5zaW9uPjIyPC9leGlmOlBpeGVsWERpbWVuc2lvbj4KICAgICAgICAgPGV4aWY6UGl4ZWxZRGltZW5zaW9uPjMwMDwvZXhpZjpQaXhlbFlEaW1lbnNpb24+CiAgICAgIDwvcmRmOkRlc2NyaXB0aW9uPgogICA8L3JkZjpSREY+CjwveDp4bXBtZXRhPgrL2/xqAAAA6UlEQVRoBe3XQQrCQAwF0Bb0Hr2heB4v14MIOtmli8GmlFHhBQbimA7h7f48TdOjnf+pua16/Z91bUqAAAECBAgQIECAAAECHwQi58Wp1KsyfPpsbLsUX133zF/a0G3PYJq5p358GxSxdaWelWGzBAgQIECAAAECBAgQIDBUIHJenDNqTHiPbZcz1m1vrPmdIyE9f5/7MeE9KKohPW+Ze+E9a+gJECBAgAABAgQIECDwHYHIeXEqNSaM9zaKbZfen537tXO/uT4S0seE8c2a6UdQVEO6MJ4AtQQIECBAgAABAgQIECDwqwJv9lMMWDcoNMAAAAAASUVORK5CYII=);
  }

  .aside-x2 {
    background-image: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABYAAAB4CAYAAAD2SgIRAAAABGdBTUEAALGPC/xhBQAAACBjSFJNAAB6JgAAgIQAAPoAAACA6AAAdTAAAOpgAAA6mAAAF3CculE8AAAAeGVYSWZNTQAqAAAACAAEARIAAwAAAAEAAQAAARoABQAAAAEAAAA+ARsABQAAAAEAAABGh2kABAAAAAEAAABOAAAAAAAAAEgAAAABAAAASAAAAAEAA6ABAAMAAAABAAEAAKACAAQAAAABAAAAFqADAAQAAAABAAAAeAAAAADVvVTBAAAACXBIWXMAAAsTAAALEwEAmpwYAAACMWlUWHRYTUw6Y29tLmFkb2JlLnhtcAAAAAAAPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyIgeDp4bXB0az0iWE1QIENvcmUgNS40LjAiPgogICA8cmRmOlJERiB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPgogICAgICA8cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0iIgogICAgICAgICAgICB4bWxuczp0aWZmPSJodHRwOi8vbnMuYWRvYmUuY29tL3RpZmYvMS4wLyIKICAgICAgICAgICAgeG1sbnM6ZXhpZj0iaHR0cDovL25zLmFkb2JlLmNvbS9leGlmLzEuMC8iPgogICAgICAgICA8dGlmZjpPcmllbnRhdGlvbj4xPC90aWZmOk9yaWVudGF0aW9uPgogICAgICAgICA8ZXhpZjpDb2xvclNwYWNlPjE8L2V4aWY6Q29sb3JTcGFjZT4KICAgICAgICAgPGV4aWY6UGl4ZWxYRGltZW5zaW9uPjIyPC9leGlmOlBpeGVsWERpbWVuc2lvbj4KICAgICAgICAgPGV4aWY6UGl4ZWxZRGltZW5zaW9uPjMwMDwvZXhpZjpQaXhlbFlEaW1lbnNpb24+CiAgICAgIDwvcmRmOkRlc2NyaXB0aW9uPgogICA8L3JkZjpSREY+CjwveDp4bXBtZXRhPgrL2/xqAAAAt0lEQVRoBe3YMQqAMAyF4VYUD+Ii7p7JK7t4EkVsnOoSEtql+AuC2GdIPxxCYwhhS3c7V0ytju20S6cIIIAAAggg8EcBGVh658YvZ75uXDqenSV3S14YFkswy5gKZ3keEUAAAQQQQACBnwnI7NZV2vNdqY5eRjqe9Ih59ciTMm2u+YuC50/hgjr6p0Ix6BHz6mlOEkQAAQQQQAABBBwCMrBwtvmCcbbp+G+IIoAAAggggAACCDgFHqn/Cpqy3VM1AAAAAElFTkSuQmCC);
  }

  .aside-x1 {
    background-image: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABYAAAA8CAYAAABig0prAAAABGdBTUEAALGPC/xhBQAAACBjSFJNAAB6JgAAgIQAAPoAAACA6AAAdTAAAOpgAAA6mAAAF3CculE8AAAAeGVYSWZNTQAqAAAACAAEARIAAwAAAAEAAQAAARoABQAAAAEAAAA+ARsABQAAAAEAAABGh2kABAAAAAEAAABOAAAAAAAAAEgAAAABAAAASAAAAAEAA6ABAAMAAAABAAEAAKACAAQAAAABAAAAFqADAAQAAAABAAAAPAAAAAB4zqpIAAAACXBIWXMAAAsTAAALEwEAmpwYAAACMWlUWHRYTUw6Y29tLmFkb2JlLnhtcAAAAAAAPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyIgeDp4bXB0az0iWE1QIENvcmUgNS40LjAiPgogICA8cmRmOlJERiB4bWxuczpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPgogICAgICA8cmRmOkRlc2NyaXB0aW9uIHJkZjphYm91dD0iIgogICAgICAgICAgICB4bWxuczp0aWZmPSJodHRwOi8vbnMuYWRvYmUuY29tL3RpZmYvMS4wLyIKICAgICAgICAgICAgeG1sbnM6ZXhpZj0iaHR0cDovL25zLmFkb2JlLmNvbS9leGlmLzEuMC8iPgogICAgICAgICA8dGlmZjpPcmllbnRhdGlvbj4xPC90aWZmOk9yaWVudGF0aW9uPgogICAgICAgICA8ZXhpZjpDb2xvclNwYWNlPjE8L2V4aWY6Q29sb3JTcGFjZT4KICAgICAgICAgPGV4aWY6UGl4ZWxYRGltZW5zaW9uPjIyPC9leGlmOlBpeGVsWERpbWVuc2lvbj4KICAgICAgICAgPGV4aWY6UGl4ZWxZRGltZW5zaW9uPjMwMDwvZXhpZjpQaXhlbFlEaW1lbnNpb24+CiAgICAgIDwvcmRmOkRlc2NyaXB0aW9uPgogICA8L3JkZjpSREY+CjwveDp4bXBtZXRhPgrL2/xqAAAArElEQVRYCe2VSwqAMAxEW3WjK3eCHsT7H0bBQ/jp7LqokIQirUyhUMIkJA8y9c65Ndx6jg+tdvW0y05J4CcEsHmtcpZTqc8rR8eLsuQm0cOAeomwGA1Q4GrOrRFTSwIkkCCArWsScUvosiSpc9DxrM5KJ+xxGLY5xIHi3xbbfBuKdvpGhnESqJ8AvGJSjnFI9LDNUSKMNKLCkT7v0/LnffO35Z2T1UiABAol8ACZ6QleECm8BQAAAABJRU5ErkJggg==);
  }

  .aside__line {
    position: absolute;
    left: 65px;
    transform: translateX(-50%);
    width: var(--size-4-1);
    height: 100%;
    background: var(--text-accent);
  }

  .filled {
    transform-origin: top center;
    z-index: 1;
    animation: scaleDown 1s ease-in-out;
    animation-fill-mode: forwards;
  }

  .filled__line__completed {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
  }

  .events {
    position: relative;
  }

  .event_item {
    margin: 0;
    cursor: pointer;
    padding: var(--size-4-1) var(--size-4-3) var(--size-4-3) 0;
    width: 100%;
    overflow: hidden;
  }

  .event_item.short {
    padding: 0;
  }

  .event_item:hover {
    background-color: var(--interactive-accent-hover);
    box-shadow: 0px 0px 52px -18px rgba(0, 0, 0, 0.75);
  }

  .event_item_color1 {
    background-color: var(--skobeloff);
  }

  .event_item_color2 {
    background-color: var(--midnight-green-eagle-green);
  }

  .event_item_color3 {
    background-color: var(--midnight-green-eagle-green-2);
  }

  .event_item_color4 {
    background-color: var(--midnight-green-eagle-green-3);
  }

  .event_item_color5 {
    background-color: var(--charcoal);
  }

  .event_item_color6 {
    background-color: var(--prussian-blue);
  }

  .event_item_color7 {
    background-color: var(--space-cadet);
  }

  .event_item_color8 {
    background-color: var(--dark-purple);
  }

  .event_item_color9 {
    background-color: var(--russian-violet);
  }

  .event_item_color10 {
    background-color: var(--russian-violet-2);
  }

  .event_item_contents {
    padding-left: 58px;
  }

  .ei_Copy,
  .ei_Title {
    color: var(--text-on-accent);
  }

  .ei_Dot,
  .ei_Title {
    display: inline-block;
  }

  .ei_Dot {
    position: absolute;
    border-radius: 50%;
    width: var(--size-4-4);
    height: var(--size-4-4);
    margin-top: var(--size-4-1);
    background-color: var(--text-accent);
    box-shadow: 0px 0px 52px -18px rgba(0, 0, 0, 0.75);
    z-index: 2;
  }

  .dot_active {
    background-color: var(--text-error-hover);
  }

  .ei_Title {
    margin-left: var(--size-4-6);
  }

  .ei_Copy {
    font-size: var(--font-ui-medium);
    display: inline-block;
    margin-left: var(--size-4-7);
  }

  .header_title,
  .ei_Title,
  .ce_title {
    color: #fff;
  }

  #now-line {
    height: var(--size-4-1);
    background-color: darkred;
    opacity: 80%;
    position: absolute;
    z-index: 3;
    width: 100%;
  }

  #now-line .timeline-time {
    position: relative;
    left: 5px;
    top: 0;
    background-color: darkred;
    color: #fff;
    padding: 0 4px 2px 4px;
    border-radius: 0 0 4px 4px;
    text-align: center;
  }

  #scroll-controls {
    position: sticky;
    bottom: 0;
    width: 100%;
    z-index: 4;
    padding: 8px 15px;
    text-align: center;
    height: 45px;
  }

  #scroll-controls label {
    display: block;
    float: left;
    margin: 2px;
  }

  #scroll-controls .toggle {
    -webkit-appearance: none;
    -moz-appearance: none;
    appearance: none;
    width: 50px;
    height: 20px;
    display: block;
    float: left;
    position: relative;
    margin-top: 5px;
    border-radius: 50px;
    overflow: hidden;
    outline: none;
    border: none;
    cursor: pointer;
    background-color: var(--background-primary);
    transition: background-color ease 0.3s;
  }

  #scroll-controls .toggle:before {
    content: "on off";
    display: block;
    position: absolute;
    z-index: 2;
    width: 17px;
    height: 17px;
    left: 2px;
    top: 1px;
    border-radius: 50%;
    font: 9px/18px Helvetica;
    text-transform: uppercase;
    font-weight: var(--font-bold);
    text-indent: -24px;
    word-spacing: 30px;
    background-color: var(--text-normal);
    text-shadow: -1px -1px rgba(0, 0, 0, 0.15);
    white-space: nowrap;
    box-shadow: 0 1px 2px rgba(0, 0, 0, 0.2);
    transition: all cubic-bezier(0.3, 1.5, 0.7, 1) 0.3s;
  }

  #scroll-controls .toggle:checked {
    background-color: var(--background-modifier-success);
  }

  #scroll-controls .toggle:checked:before {
    left: 31px;
  }

  .empty-timeline {
    text-align: center;
    vertical-align: middle;
    margin-top: 50%;
  }
</style>
