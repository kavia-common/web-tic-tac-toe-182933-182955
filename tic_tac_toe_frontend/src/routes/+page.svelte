<script lang="ts">
    // Game state and logic for Tic Tac Toe
    type Player = 'X' | 'O';
    type Cell = Player | '';

    const initialBoard: Cell[] = Array(9).fill('');
    let board: Cell[] = [...initialBoard];
    let currentPlayer: Player = 'X';
    let winner: Player | null = null;
    let isDraw = false;
    let winningLine: number[] = [];
    let gameOver = false;

    const lines: number[][] = [
        [0, 1, 2],
        [3, 4, 5],
        [6, 7, 8],
        [0, 3, 6],
        [1, 4, 7],
        [2, 5, 8],
        [0, 4, 8],
        [2, 4, 6]
    ];

    function checkWinner(b: Cell[]): { winner: Player | null; line: number[]; draw: boolean } {
        for (const [a, c, d] of lines) {
            if (b[a] && b[a] === b[c] && b[a] === b[d]) {
                return { winner: b[a] as Player, line: [a, c, d], draw: false };
            }
        }
        const draw = b.every((v) => v !== '');
        return { winner: null, line: [], draw };
    }

    // PUBLIC_INTERFACE
    function resetGame() {
        /** Reset the game to initial state. */
        board = [...initialBoard];
        currentPlayer = 'X';
        winner = null;
        isDraw = false;
        winningLine = [];
        gameOver = false;
    }

    // PUBLIC_INTERFACE
    function handleCellClick(index: number) {
        /** Handle a player's move by placing X or O in an empty cell and checking game status. */
        if (gameOver || board[index] !== '') return;
        board[index] = currentPlayer;

        const { winner: w, line, draw } = checkWinner(board);
        if (w) {
            winner = w;
            winningLine = line;
            gameOver = true;
        } else if (draw) {
            isDraw = true;
            gameOver = true;
        } else {
            currentPlayer = currentPlayer === 'X' ? 'O' : 'X';
        }
    }

    function isWinningIndex(i: number) {
        return winningLine.includes(i);
    }

    // Map player -> Unicode chess glyph and accessible names
    function glyphFor(cell: Cell): string {
        // Prefer white pieces for clarity
        if (cell === 'X') return '♘'; // Knight
        if (cell === 'O') return '♕'; // Queen
        return '';
    }
    function ariaFor(cell: Cell): string {
        if (cell === 'X') return 'knight';
        if (cell === 'O') return 'queen';
        return 'empty';
    }
</script>

<svelte:head>
    <title>Tic Tac Toe — Ocean Professional</title>
    <meta name="description" content="Play classic Tic Tac Toe in a modern, accessible Svelte app." />
</svelte:head>

<div class="page">
    <div class="card" role="region" aria-label="Tic Tac Toe game">
        <header class="header">
            <h1 class="title">Tic Tac Toe</h1>
            <p class="subtitle">Ocean Professional</p>
        </header>

        <section class="status" aria-live="polite" aria-atomic="true">
            {#if winner}
                <span class="badge badge-win" data-testid="status-winner">Player {winner} wins!</span>
            {:else if isDraw}
                <span class="badge badge-draw" data-testid="status-draw">It's a draw.</span>
            {:else}
                <span class="turn" data-testid="status-turn">
                    <span class="dot" aria-hidden="true"></span>
                    Player <strong class="turn-player">{currentPlayer}</strong>'s turn
                </span>
            {/if}
        </section>

        <section class="board-wrapper">
            <div
                class="board"
                style="--cells: 3"
                role="grid"
                aria-label="3 by 3 Tic Tac Toe board"
            >
                {#each board as cell, i (i)}
                    <button
                        class="cell {cell ? 'filled' : ''} {isWinningIndex(i) ? 'win' : ''}"
                        role="gridcell"
                        aria-label={`Cell ${i + 1}, ${ariaFor(cell)}`}
                        aria-disabled={gameOver || cell !== ''}
                        disabled={gameOver || cell !== ''}
                        on:click={() => handleCellClick(i)}
                    >
                        <span
                            class="mark {cell === 'X' ? 'player-x' : ''} {cell === 'O' ? 'player-o' : ''}"
                            data-testid={`cell-${i}`}
                            aria-hidden={cell === ''}
                        >
                            {glyphFor(cell)}
                        </span>
                    </button>
                {/each}
            </div>
        </section>

        <div class="actions">
            <button
                class="btn primary"
                on:click={resetGame}
                aria-label="Start a new game"
                data-testid="reset-button"
            >
                New Game
            </button>
        </div>
    </div>

    <footer class="footer-hint">
        <small>Tip: Click any empty cell to place your mark. X starts.</small>
    </footer>
</div>

<style>
    :root {
        --primary: #2563EB;      /* blue */
        --secondary: #F59E0B;    /* amber */
        --error: #EF4444;
        --bg: #f9fafb;
        --surface: #ffffff;
        --text: #111827;
        --muted: #6b7280;

        --shadow-sm: 0 1px 2px rgba(0,0,0,0.06), 0 1px 1px rgba(0,0,0,0.04);
        --shadow-md: 0 8px 24px rgba(0, 0, 0, 0.08);
        --radius: 14px;
        --radius-sm: 10px;

        --cell-size: min(22vw, 110px);
        --cell-gap: 10px;
        --transition-fast: 140ms cubic-bezier(.2,.8,.2,1);
        --transition: 220ms cubic-bezier(.2,.8,.2,1);
    }

    .page {
        min-height: calc(100dvh - 0px);
        padding: 24px;
        background: linear-gradient(145deg, rgba(59,130,246,0.08), rgba(249,250,251,0.9));
        display: grid;
        place-items: center;
        color: var(--text);
    }

    .card {
        background: var(--surface);
        border-radius: var(--radius);
        box-shadow: var(--shadow-md);
        padding: 22px;
        width: min(92vw, 540px);
        display: flex;
        flex-direction: column;
        gap: 18px;
        border: 1px solid rgba(17,24,39,0.06);
    }

    .header {
        display: flex;
        align-items: baseline;
        justify-content: space-between;
        gap: 12px;
        border-bottom: 1px solid rgba(17,24,39,0.06);
        padding-bottom: 10px;
    }
    .title {
        margin: 0;
        font-size: 1.35rem;
        font-weight: 700;
        letter-spacing: .2px;
        color: var(--text);
    }
    .subtitle {
        margin: 0;
        color: var(--muted);
        font-size: .85rem;
    }

    .status {
        display: flex;
        justify-content: center;
        align-items: center;
        min-height: 36px;
    }

    .turn {
        display: inline-flex;
        align-items: center;
        gap: 8px;
        color: var(--muted);
        font-weight: 500;
    }
    .turn-player {
        color: var(--primary);
        font-weight: 800;
        letter-spacing: .3px;
    }
    .dot {
        width: 8px;
        height: 8px;
        border-radius: 999px;
        background: var(--secondary);
        box-shadow: 0 0 0 6px rgba(245, 158, 11, 0.12);
        animation: pulse 1.8s ease-in-out infinite;
    }
    @keyframes pulse {
        0%, 100% { transform: scale(1); opacity: 1; }
        50% { transform: scale(1.18); opacity: .85; }
    }

    .badge {
        display: inline-flex;
        align-items: center;
        gap: 8px;
        font-weight: 700;
        letter-spacing: .3px;
        padding: 8px 12px;
        border-radius: 999px;
        box-shadow: var(--shadow-sm);
        border: 1px solid transparent;
    }
    .badge-win {
        color: var(--surface);
        background: linear-gradient(180deg, var(--primary), #1e40af);
        border-color: rgba(37,99,235,0.25);
    }
    .badge-draw {
        color: #92400e;
        background: linear-gradient(180deg, #fde68a, #fbbf24);
        border-color: rgba(245,158,11,0.35);
    }

    .board-wrapper {
        display: grid;
        place-items: center;
    }

    .board {
        display: grid;
        grid-template-columns: repeat(var(--cells, 3), var(--cell-size));
        grid-template-rows: repeat(var(--cells, 3), var(--cell-size));
        gap: var(--cell-gap);
        padding: 12px;
        border-radius: var(--radius);
        background: linear-gradient(180deg, rgba(255,255,255,0.9), rgba(243,244,246,0.8));
        box-shadow: inset 0 1px 0 rgba(255,255,255,0.6), var(--shadow-sm);
        border: 1px solid rgba(17,24,39,0.06);
        transition: transform var(--transition);
        will-change: transform;
    }
    .board:focus-within {
        transform: translateY(-1px);
    }

    .cell {
        position: relative;
        width: var(--cell-size);
        height: var(--cell-size);
        border-radius: var(--radius-sm);
        border: 1px solid rgba(17,24,39,0.08);
        background: #ffffff;
        display: grid;
        place-items: center;
        cursor: pointer;
        transition: transform var(--transition-fast), box-shadow var(--transition-fast), border-color var(--transition-fast), background var(--transition-fast);
        box-shadow: var(--shadow-sm);
        outline: none;
    }
    .cell:hover:not(:disabled) {
        transform: translateY(-2px);
        box-shadow: 0 6px 18px rgba(0,0,0,0.08);
        border-color: rgba(37,99,235,0.35);
    }
    .cell:active:not(:disabled) {
        transform: translateY(0);
    }
    .cell:disabled {
        cursor: default;
        opacity: 1;
    }
    .cell.filled {
        background: linear-gradient(180deg, #ffffff, #f3f4f6);
    }

    .cell.win {
        background: linear-gradient(180deg, #dbeafe, #bfdbfe);
        border-color: rgba(37,99,235,0.6);
        box-shadow: 0 0 0 3px rgba(37,99,235,0.22), 0 10px 24px rgba(37,99,235,0.25);
        transform: translateY(-2px) scale(1.02);
    }

    /* Mark now represents a chess icon. Keep sizing responsive and theme-aware. */
    .mark {
        font-weight: 900;
        font-size: calc(var(--cell-size) * 0.48);
        line-height: 1;
        color: var(--text);
        transition: transform var(--transition), color var(--transition);
        user-select: none;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
    }

    /* Smooth transition for empty cells */
    .cell:not(.filled) .mark {
        color: transparent;
    }

    .cell.filled .mark {
        color: var(--text);
    }

    /* Differentiate players with subtle theme accents for better contrast */
    .cell.filled .mark.player-x { /* Knight */
        color: var(--text);
        text-shadow: 0 1px 0 rgba(255,255,255,0.6);
    }
    .cell.filled .mark.player-o { /* Queen */
        color: var(--primary);
        text-shadow: 0 1px 0 rgba(255,255,255,0.6);
    }

    .actions {
        display: flex;
        justify-content: center;
        padding-top: 6px;
    }

    .btn {
        appearance: none;
        border: 1px solid rgba(17,24,39,0.1);
        background: var(--surface);
        color: var(--text);
        padding: 10px 16px;
        border-radius: 12px;
        font-weight: 700;
        letter-spacing: .2px;
        box-shadow: var(--shadow-sm);
        transition: transform var(--transition-fast), box-shadow var(--transition-fast), background var(--transition-fast), color var(--transition-fast), border-color var(--transition-fast);
        cursor: pointer;
        outline: none;
    }
    .btn:hover {
        transform: translateY(-2px);
        box-shadow: 0 8px 22px rgba(0,0,0,0.08);
    }
    .btn:active {
        transform: translateY(0px);
        box-shadow: var(--shadow-sm);
    }
    .btn.primary {
        background: linear-gradient(180deg, var(--primary), #1d4ed8);
        color: #fff;
        border-color: rgba(37,99,235,0.3);
    }
    .btn.primary:hover {
        box-shadow: 0 8px 26px rgba(37,99,235,0.35);
    }
    .btn.primary:focus-visible {
        outline: 3px solid rgba(245, 158, 11, 0.35);
        outline-offset: 2px;
    }

    .footer-hint {
        margin-top: 14px;
        color: var(--muted);
        text-align: center;
    }

    /* Responsive adjustments */
    @media (max-width: 420px) {
        :root {
            --cell-size: min(22vw, 92px);
        }
        .card {
            padding: 18px;
        }
    }
</style>
