<script>
    import { onMount } from "svelte";
    import Cell from "../components/Cell.svelte";
    import { puzzle } from "../stores";
    /**
    * @type { Array<Array<{number: Number, isInput: boolean}>> }
    */
    let puzzleGrid = [[]];
    let playerWin = false;
    let playerLose = false;
    let giveUp = false;
    /** 
    * @type { Array<Array<Number>> }
    */
    let grid = [
        [5, 3, 9, 4, 7, 6, 8, 1, 2],
        [2, 4, 1, 9, 5, 8, 6, 3, 7],
        [6, 8, 7, 1, 2, 3, 9, 4, 5],
        [9, 1, 3, 5, 6, 7, 2, 8, 4],
        [7, 6, 4, 2, 8, 1, 5, 9, 3],
        [8, 2, 5, 3, 9, 4, 7, 6, 1],
        [3, 7, 2, 6, 4, 9, 1, 5, 8],
        [4, 5, 6, 8, 1, 2, 3, 7, 9],
        [1, 9, 8, 7, 3, 5, 4, 2, 6]
    ]
    puzzle.subscribe(value => {
        puzzleGrid = value;
        let winCount = 0;
        if(puzzleGrid.length && !giveUp) {
            for (let i = 0; i < grid.length; i++) {
                for (let j = 0; j < grid.length; j++) {
                    if(grid[i][j] == puzzleGrid[i][j].number) {
                        winCount++;
                    }
                }
            }
            if(winCount == 81) {
                console.log("WIN");
                playerWin = true;
            }
        }
    });

    /**
     * @param {number[]} arr
     * @param {number} i1
     * @param {number} i2
     */
    function swapElements(arr, i1, i2) {
        // Step 1
        let temp = arr[i1];

        // Step 2
        arr[i1] = arr[i2];

        // Step 3
        arr[i2] = temp;
    }
     
    function generate9() {
        for (let i = 0; i < 20; i++) {
            let min = 1;
            let max = 9;
            let random1 = Math.floor(Math.random() * (max - min + 1) + min);
            let random2 = Math.floor(Math.random() * (max - min + 1) + min);
            while(random1 == random2) {
                random2 = Math.floor(Math.random() * (max - min + 1) + min);
                //console.log("random was the same, restarting...");
            }
            //console.log(random1 + " different from " + random2);

            for (let j = 0; j < grid.length; j++) {
                //console.log("==============================");
                //console.log(grid[j]);
                let index1 = grid[j].indexOf(random1);
                let index2 = grid[j].indexOf(random2);
                //console.log(index1 + " has " + random1);
                //console.log(index2 + " has " + random2);
                swapElements(grid[j], index1, index2);
                //console.log(grid[j]);
            }
        }
        console.log(grid);
    }
    generate9();

    async function populatePuzzle() {
        /**
         * @type {Array<Array<{number: Number, isInput: boolean}>>}
         */
        let newPuzzle = [[], [], [],[], [], [],[], [], [],];
        let rowCount = 0;
        let colCount = 0;
        for (const row of grid) {
            for (const col of row) {
                let rand = Math.floor(Math.random() * 4);
                if(rand == 1) {
                    newPuzzle[rowCount].push({
                        number: 0,
                        isInput: true
                    })
                }
                else {
                    newPuzzle[rowCount].push({
                        number: col,
                        isInput: false
                    })
                }
            }
            rowCount++;
        }
        return newPuzzle;
    }   
    
    async function generatePuzzle() {
        let myPuzzle = await populatePuzzle();
        // @ts-ignore
        puzzle.update(value => myPuzzle);
        return myPuzzle;
    }


    

    /**
     * @param {Array<Array<{number: Number, isInput: boolean}>>} board
     * @param {number} row
     * @param {number} number
     */
    function numberIsAlreadyInRow(board, row, number){
        for(let i = 0 ; i < grid.length ; i++){
            if(board[row][i].number == number ) {
                return true;
            }
        }
        return false;
    }
    /**
     * @param {Array<Array<{number: Number, isInput: boolean}>>} board
     * @param {number} column
     * @param {number} number
     */
    function numberIsAlreadyInColumn(board, column, number){
        for(let j = 0 ; j < grid.length ; j++){
            if(board[j][column].number == number ) {
                return true;
            }
        }
        return false;
    }
    /**
     * @param {Array<Array<{number: Number, isInput: boolean}>>} board
     * @param {number} column
     * @param {number} row
     * @param {number} number
     */
    function numberIsAlreadyInBox(board, row, column, number){
        let SpecificBoxRow = row - row % 3;
        let SpecificBoxColumn = column - column % 3;

        for(let i = SpecificBoxRow ; i < SpecificBoxRow + 3 ; i++){
            for(let j = SpecificBoxColumn; j < SpecificBoxColumn + 3 ; j++){
                if(board[i][j].number == number){
                    return true;
                }
            }
        }
        return false;
    }
    /**
     * @param {Array<Array<{number: Number, isInput: boolean}>>} board
     * @param {number} column
     * @param {number} row
     * @param {number} number
     */
    function isValidNumber(board, row , column , number) {
        return !numberIsAlreadyInRow(board,row,number) && !numberIsAlreadyInColumn(board,column,number)
                && !numberIsAlreadyInBox(board,row,column,number);
    }
    function resetAnswer() {
        for (let i = 0; i < grid.length; i++) {
            for (let j = 0; j < grid.length; j++) {
                if(puzzleGrid[i][j].isInput) {
                    puzzleGrid[i][j].number = 0;
                }
            }
        }
    }
    function solutionBoard(){
        giveUp = true;
        for(let row = 0; row < grid.length;row++){
            for(let column = 0 ; column < grid.length; column++) {
                if (puzzleGrid[row][column].number == 0) {
                    for (let possibleNumber = 1; possibleNumber <= grid.length; possibleNumber++) {
                        if (isValidNumber(puzzleGrid, row, column, possibleNumber)) {
                            puzzleGrid[row][column].number = possibleNumber;
                            if (solutionBoard()) {
                                return true;
                            }
                            else puzzleGrid[row][column].number = 0;
                        }
                    }
                    return false;
                }
            }
        }
        console.log(puzzleGrid);
        // @ts-ignore
        puzzle.update(value => puzzleGrid);
        playerLose = true;
        return true;
    }
</script>

<section>
    {#if playerWin}
        <div class="win">
            <h1>You Win!</h1>
            <button on:click={() => location.reload()}>Try again?</button>
        </div>
    {/if}
    {#if playerLose}
        <div class="lose">
            <h1>You Lose!</h1>
            <button on:click={() => location.reload()}>Try again?</button>
        </div>
    {/if}
</section>
<main>
    {#if !giveUp}
        {#await generatePuzzle()}
        <h1>Loading</h1>
        {:then showGrid}
        <div class="grid">
        {#each Object.values(showGrid) as row, i}
            {#each row as col, j}
                <div class="cell">
                    <Cell cellData={{ 
                        row: i,
                        col: j,
                        number: col.number,
                        isInput: col.isInput
                    }}/>
                </div>
            {/each}
        {/each}
        <button class="giveUp" on:click={() => {{
            resetAnswer();
            solutionBoard();
        }}}>Give up</button>
    </div>
    {/await}
    {:else}
        <div class="grid">
            {#each puzzleGrid as row, i}
                {#each row as col, j}
                    <div class="cell">
                        <p class:wasInput={col.isInput}>{col.number}</p>
                    </div>
                {/each}
            {/each}
        </div>
    {/if}
</main>
<style>
    .wasInput {
        width: 100%;
        display: grid;
        place-items: center;
        padding: 0;
        margin: 0;
        height: 100%;
        border: none;
        font-size: 2.5rem;
        background-color: rgb(0, 255, 255);
        color: rgb(255, 97, 229);
        text-shadow: 0 0 5px rgb(248, 133, 229);
        filter: drop-shadow(0 0 3px rgb(0, 255, 255));
        line-height: 1;
    }
    .giveUp {
        position: absolute;
        bottom: 0;
        left: 50%;
        margin-bottom: 5rem;
        transform: translateX(-50%);
        background-color: transparent;
        border: none;
        font-size: 4rem;
        color: red;
        text-shadow: 0 0 5px red;
        transition: 0.25s ease-in-out;
        cursor: pointer;
    }
    .giveUp:hover {
        font-size: 5.5rem;
        text-shadow: 0 0 10px red;
    }
    .win {
        position: absolute;
        top: 50%;
        left: 15%;
        transform: translate(-15%, -50%);
    }
    .win h1{
        font-size: 5rem;
        color: rgb(1, 233, 1);
        text-shadow: 0 0 8px rgb(0, 255, 0);
    }
    .win button{
        background: transparent;
        font-size: 3rem;
        color: aqua;
        text-shadow: 0 0 8px aqua;
    }
    .lose {
        position: absolute;
        top: 50%;
        left: 15%;
        transform: translate(-15%, -50%);
    }
    .lose h1{
        font-size: 5rem;
        color: rgb(233, 1, 51);
        text-shadow: 0 0 8px rgb(233, 1, 51);
    }
    .lose button{
        background: transparent;
        font-size: 3rem;
        color: aqua;
        text-shadow: 0 0 8px aqua;
    }
    main {
        display: grid;
        place-items: center;
        height: 100vh;
        background-image: url('https://media4.giphy.com/media/3o6vXTpomeZEyxufGU/giphy.gif?cid=790b761124b46867a0914972a7fb9385024012ebe6ade742&rid=giphy.gif&ct=g');
        background-repeat: no-repeat;
        background-size: 900px;
        background-position: center;
    }
    .grid {
        display: grid;
        grid-template-columns: repeat(9, 5rem);
    }
    .cell {
        height: 5rem;
        border: 1px solid rgb(255, 90, 233);
        display: grid;
        place-items: center;
        background-color: rgba(0, 0, 0, 0.5);
    }
    p {
        font-size: 2rem;
    }
</style>