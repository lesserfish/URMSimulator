<script>
    export let command;
    export let ID;
    export let deleteButton;
    export let selectButton;
    export let currentStep;
    function getBackgroundColor()
    {
        if(currentStep == ID)
        {
            return "bg-primary text-secondary "
        }
        else if(ID % 2 == 0){
            return "bg-secondary text-gray"
        }
        else{
            return "bg-gray text-gray"
        }
    }
</script>
<div class ="columns text-center cmdcolumns">
    <div class="column col">
        <button class="btn btn-link col-8 {getBackgroundColor(currentStep)} text-left" 
        style="outline: none;" on:click={() => {selectButton(ID)}}>
            {command.string}
        </button>
        <button class="btn btn-action btnCol {getBackgroundColor(currentStep)}" 
        style="font-size: 140%;" on:click={() => {deleteButton(ID)}}>
        <i class="icon icon-cross"></i>
        </button>
</div>
</div>

<style>
    .cmdcolumns{
        margin-left: 25%;
        margin-right: 25%;
    }
    .firstCol{
        background-color: rgb(162, 168, 173);
    }
    .btnCol{
        margin: 5px;

    }
</style>