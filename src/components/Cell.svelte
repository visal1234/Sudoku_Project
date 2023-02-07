<script>
    import { puzzle } from "../stores";
    /**
    * @type { Array<Array<{number: Number, isInput: boolean}>> }
    */
    let puzzleGrid;
    puzzle.subscribe(value => {
        puzzleGrid = value
    });

    export let cellData = {
        row: 0,
        col: 0,
        number: 0,
        isInput: false
    }

    /**
     * @param {number} num
     */
    function setAnswer(num) {
        if(myAnswer == num && !cellData.isInput) return;
        puzzleGrid[cellData.row][cellData.col].number = num;
        // @ts-ignore
        puzzle.update(value => puzzleGrid);
    }
    /**
     * @type {HTMLInputElement}
     */
    let myInput;
    let myAnswer = 0;

    $: setAnswer(cellData.number);
</script>

{#if cellData.isInput}
    <input type="number" bind:value={cellData.number} bind:this={myInput} on:click={() => myInput.select()}>
{:else}
   <p>{cellData.number}</p>
{/if}

<style>
    p {
        font-size: 2rem;
    }
    input {
        width: 100%;
        text-align: center;
        padding: 0;
        margin: 0;
        height: 100%;
        border: none;
        font-size: 2.5rem;
        background-color: rgb(0, 255, 255);
        color: rgb(165, 97, 255);
        text-shadow: 0 0 5px rgb(196, 151, 255);
        filter: drop-shadow(0 0 3px rgb(0, 255, 255));
    }
    input:focus {
        outline: 3px solid rgb(0, 255, 242);
    }
    input::-webkit-inner-spin-button {
        -webkit-appearance: none;
        margin: 0;
    }
    /* Firefox */
    input[type=number] {
    -moz-appearance: textfield;
    }
</style>
