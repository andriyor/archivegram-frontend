<script>
	import Textfield, {Input, Textarea} from '@smui/textfield';
	import Icon from '@smui/textfield/icon';
	let valueStandardB = '';
</script>

<style>

</style>


<Textfield withLeadingIcon bind:value={valueStandardB} label="Username" input$aria-controls="helper-text-standard-b" input$aria-describedby="helper-text-standard-b">
	<Icon class="material-icons">playlist_add</Icon>
</Textfield>