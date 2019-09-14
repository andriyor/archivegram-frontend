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
  });

  function fetchUsers() {
    return fetch("http://localhost:3000/users", { method: "GET", mode: "cors" })
      .then(response => response.json())
      .then(myJson => myJson["users"]);
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
    sendUser(instagramUser).then(user => subscribedInstagramUser = [user, ...subscribedInstagramUser]);
    instagramUsersSearchResult = [];
    searchUserName = "";
  }

  function removeUser(userPk) {
    fetch(`http://localhost:3000/users/${userPk}`, { method: "DELETE" });
    subscribedInstagramUser = subscribedInstagramUser.filter(
      user => user.pk !== userPk
    );
  }
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
          remove
        </Meta>
      </Item>
    {/each}
  </List>
{/if}
