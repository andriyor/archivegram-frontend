<script>
  import { onMount } from "svelte";
  import debounce from "lodash/debounce";

  import List, {
    Item,
    Graphic,
    Meta,
    Text,
    PrimaryText,
    SecondaryText
  } from "@smui/list";
  import Textfield, { Input, Textarea } from "@smui/textfield";
  import Button, { Label } from "@smui/button";
  import Icon from "@smui/textfield/icon";

  let searchUserName = "";
  let subscribedInstagramUser = [];
  let instagramUsersSearchResult = [];

  onMount(() => {
    fetchUsers().then(
      subscribedUsers => (subscribedInstagramUser = subscribedUsers)
    );
    fetchStories().then(st => {
      st.forEach(element => {
         stories.update(element);   
      });
    });
  });

  function fetchUsers() {
    return fetch("http://localhost:3000/users", { method: "GET", mode: "cors" })
      .then(response => response.json())
      .then(myJson => myJson["users"]);
  }

  function fetchStories() {
    return fetch("http://localhost:3000/stories", {
      method: "GET",
      mode: "cors"
    }).then(response => response.json());
  }

  function sendUser(username) {
    return fetch("http://localhost:3000/users", {
      method: "POST",
      body: JSON.stringify(username),
      headers: {
        "Content-Type": "application/json"
      }
    }).then(response => response.json());
  }

  function serchIntagramUser(username) {
    return fetch(`http://localhost:3000/instagram-user?username=${username}`)
      .then(response => response.json())
      .then(myJson => myJson["users"]);
  }

  const handleInput = debounce(event => {
    let username = event.target.value;
    if (username === "") {
      instagramUsersSearchResult = [];
    } else {
      serchIntagramUser(username).then(
        usernames => (instagramUsersSearchResult = usernames)
      );
    }
  }, 500);

  function selectInstagramUser(instagramUser) {
    sendUser(instagramUser).then(
      user => (subscribedInstagramUser = [user, ...subscribedInstagramUser])
    );
    instagramUsersSearchResult = [];
    searchUserName = "";
  }

  function removeUser(userPk) {
    fetch(`http://localhost:3000/users/${userPk}`, { method: "DELETE" });
    subscribedInstagramUser = subscribedInstagramUser.filter(
      user => user.pk !== userPk
    );
  }

  let stories = new Zuck("story", {
    skin: "snapgram", // container class
    avatars: true, // shows user photo instead of last story item preview
    list: false, // displays a timeline instead of carousel
    openEffect: true, // enables effect when opening story - may decrease performance
    cubeEffect: false, // enables the 3d cube effect when sliding story - may decrease performance
    autoFullScreen: false, // enables fullscreen on mobile browsers
    backButton: true, // adds a back button to close the story viewer
    backNative: false, // uses window history to enable back button on browsers/android
    previousTap: true, // use 1/3 of the screen to navigate to previous item when tap the story
    localStorage: true, // set true to save "seen" position. Element must have a id to save properly.
    reactive: true, // set true if you use frameworks like React to control the timeline (see react.sample.html)

    stories: [],

    callbacks: {
      onOpen(storyId, callback) {
        callback(); // on open story viewer
      },

      onView(storyId) {
        // on view story
      },

      onEnd(storyId, callback) {
        callback(); // on end story
      },

      onClose(storyId, callback) {
        callback(); // on close story viewer
      },

      onNavigateItem(storyId, nextStoryId, callback) {
        callback(); // on navigate item of story
      },

      onDataUpdate(currentState, callback) {
        callback(); // use to update state on your reactive framework
      }
    },

    template: {
      // use these functions to render custom templates
      // see src/zuck.js for more details

      timelineItem(itemData) {
        return ``;
      },

      timelineStoryItem(itemData) {
        return ``;
      },

      viewerItem(storyData, currentStoryItem) {
        return ``;
      },

      viewerItemPointer(index, currentIndex, item) {
        return ``;
      },

      viewerItemBody(index, currentIndex, item) {
        return ``;
      }
    },

    language: {
      // if you need to translate :)
      unmute: "Touch to unmute",
      keyboardTip: "Press space to see next",
      visitLink: "Visit link",
      time: {
        ago: "ago",
        hour: "hour",
        hours: "hours",
        minute: "minute",
        minutes: "minutes",
        fromnow: "from now",
        seconds: "seconds",
        yesterday: "yesterday",
        tomorrow: "tomorrow",
        days: "days"
      }
    }
  });
</script>

<style>
  * :global(.avatarsearch) {
    height: 40px;
  }
  * :global(.search-list) {
    height: 150px;
    overflow: auto;
  }
  * :global(.search-list-wrapper) {
    width: 30%;
  }
</style>

<Textfield
  withLeadingIcon
  on:input={handleInput}
  bind:value={searchUserName}
  label="Username"
  input$aria-controls="helper-text-standard-b"
  input$aria-describedby="helper-text-standard-b">
  <Icon class="material-icons">playlist_add</Icon>
</Textfield>

{#if instagramUsersSearchResult.length}
  <div class="search-list-wrapper">
    <List class="search-list" dense avatarList>
      {#each instagramUsersSearchResult as instagramUser}
        <Item
          class="avatarsearch"
          on:SMUI:action={() => selectInstagramUser(instagramUser)}>
          <Graphic
            style="background-image: url({instagramUser.profile_pic_url});
            background-size: 35px;" />
          <Text>{instagramUser.full_name}</Text>
        </Item>
      {/each}
    </List>
  </div>
{/if}

{#if subscribedInstagramUser.length}
  <List class="demo-list" twoLine avatarList singleSelection>
    {#each subscribedInstagramUser as subscribedUser}
      <Item>
        <Graphic
          style="background-image: url({subscribedUser.profile_pic_url}); ;
          background-size: 40px;" />
        <Text>
          <PrimaryText>{subscribedUser.full_name}</PrimaryText>
          <SecondaryText>{subscribedUser.username}</SecondaryText>
        </Text>
        <Meta
          class="material-icons"
          on:click={() => removeUser(subscribedUser.pk)}>
          clear
        </Meta>
      </Item>
    {/each}
  </List>
{/if}
