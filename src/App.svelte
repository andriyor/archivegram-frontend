<script>
  import { onMount } from "svelte";

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

  let userName = "";

  let optionsTwoLine = [];

  onMount(() => {
    fetchUsers();
  });

  function fetchUsers() {
    fetch("http://localhost:3000/users", { method: "GET", mode: "cors" })
      .then(function(response) {
        return response.json();
      })
      .then(function(myJson) {
        optionsTwoLine = myJson["users"];
        console.log(myJson);
      });
  }

  function sendUsername() {
    console.log(userName);
    fetch("http://localhost:3000/users", {
      method: "POST", // or 'PUT'
      body: JSON.stringify({ user: { username: userName } }), // data can be `string` or {object}!
      headers: {
        "Content-Type": "application/json"
      }
    });
  }

  function handlePress(event) {
    console.log("kek");
    if (event.key === "Enter") {
      sendUsername();
    }
  }
</script>

<style>

</style>

<Textfield
  withLeadingIcon
  bind:value={userName}
  on:keydown={handlePress}
  label="Username"
  input$aria-controls="helper-text-standard-b"
  input$aria-describedby="helper-text-standard-b">
  <Icon class="material-icons">playlist_add</Icon>
</Textfield>
<Button on:click={sendUsername}>
  <Icon class="material-icons">add</Icon>
  <Label>Add user</Label>
</Button>

<List class="demo-list" twoLine avatarList singleSelection>
  {#each optionsTwoLine as item}
    <Item>
      <Graphic
        style="background-image: url(https://via.placeholder.com/40x40.png?text={item.fullname
          .split(' ')
          .map(val => val.substring(0, 1))
          .join('')});" />
      <Text>
        <PrimaryText>{item.fullname}</PrimaryText>
        <SecondaryText>{item.username}</SecondaryText>
      </Text>
      <Meta class="material-icons">info</Meta>
    </Item>
  {/each}
</List>
