<script lang="ts">
	import Gameover from './gameover.svelte';
	import Tile from './tile.svelte';

	const SIZE = 9;
	let bombsPlaced = false;
	let gameoverMessage = $state('');
	let gameoverOpen = $state(false);

	interface Tile {
		value: number | string;
		visible: boolean;
		flagged: boolean;
	}
	type Board = Tile[];
	let tiles: Board = $state(createEmptyBoard());

	function createEmptyBoard(): Board {
		let data: Board = [];
		for (let i = 0; i < SIZE; i++) {
			for (let j = 0; j < SIZE; j++) {
				data.push({ value: 0, visible: false, flagged: false });
			}
		}
		return data;
	}

	function setupBombs(index: number) {
		let bombCount = 10;
		while (bombCount > 0) {
			const placementIndex = Math.floor(Math.random() * SIZE * SIZE);
			if (tiles[placementIndex].value === '💣' || toCloseToStart(placementIndex, index)) {
				continue;
			}
			tiles[placementIndex].value = '💣';
			bombCount -= 1;
		}

		updateBombCounts();
	}

	function toCloseToStart(bombIndex: number, startIndex: number): boolean {
		const bombLocation = getTileRowCol(bombIndex);
		const startLocation = getTileRowCol(startIndex);
		const rowDist = Math.abs(bombLocation.row - startLocation.row);
		const colDist = Math.abs(bombLocation.col - startLocation.col);
		return rowDist <= 1 && colDist <= 1;
	}

	function getTileRowCol(index: number) {
		return {
			row: index % SIZE,
			col: Math.floor(index / SIZE)
		};
	}

	function getTileIndex(row: number, col: number): number {
		return row * SIZE + col;
	}

	function isBombAt(row: number, col: number) {
		if (row < 0 || row >= SIZE || col < 0 || col >= SIZE) {
			return false;
		}

		let index = getTileIndex(row, col);
		if (index < 0 || index >= tiles.length) {
			return false;
		}
		return tiles[index].value === '💣';
	}

	function updateBombCounts() {
		for (let row = 0; row < SIZE; row++) {
			for (let col = 0; col < SIZE; col++) {
				const tileIndex = getTileIndex(row, col);
				if (tiles[tileIndex].value === '💣') {
					continue;
				}
				const count = getTileBombCount(row, col);
				tiles[tileIndex].value = count;
			}
		}
	}

	function getTileBombCount(row: number, col: number) {
		let count = 0;
		const rowOffsets = [-1, 0, 1];
		const colOffsets = [-1, 0, 1];
		for (let deltaRow of rowOffsets) {
			for (let deltaCol of colOffsets) {
				if (isBombAt(row + deltaRow, col + deltaCol)) {
					count++;
				}
			}
		}
		return count;
	}

	function floodFill(row: number, col: number, seen: Set<number>) {
		if (row < 0 || row >= SIZE || col < 0 || col >= SIZE || seen.has(getTileIndex(row, col))) {
			return;
		}
		seen.add(getTileIndex(row, col));
		tiles[getTileIndex(row, col)].visible = true;
		if (tiles[getTileIndex(row, col)].value === 0) {
			// Up
			floodFill(row + 1, col, seen);
			// Down
			floodFill(row - 1, col, seen);
			// Left
			floodFill(row, col - 1, seen);
			// Right
			floodFill(row, col + 1, seen);
			// Diags
			floodFill(row + 1, col + 1, seen);
			floodFill(row + 1, col - 1, seen);
			floodFill(row - 1, col - 1, seen);
			floodFill(row - 1, col + 1, seen);
		} else {
			return;
		}
	}

	function tileClicked(index: number) {
		if (!bombsPlaced) {
			setupBombs(index);
			bombsPlaced = true;
		}
		tiles[index].visible = true;

		const row = Math.floor(index / SIZE);
		const col = index % SIZE;
		const seen: Set<number> = new Set();
		floodFill(row, col, seen);

		const clickedBomb = tiles[index].value === '💣';
		if (clickedBomb) {
			gameoverMessage = 'You Lose!';
			gameoverOpen = true;
		} else if (gameWon()) {
			gameoverMessage = 'Winner!';
			gameoverOpen = true;
		}
	}

	function gameWon() {
		return tiles.filter((tile) => tile.value !== '💣' && !tile.visible).length === 0;
	}

	function resetBoard() {
		tiles = createEmptyBoard();
		bombsPlaced = false;
	}

	function flagTile(index: number) {
		const tile = tiles[index];
		tile.flagged = !tile.flagged;
		tiles = tiles; // trigger update
	}

	function handleContextMenu(event: MouseEvent, index: number) {
		event.preventDefault();
		flagTile(index);
	}
</script>

<Gameover message={gameoverMessage} bind:open={gameoverOpen} reset={() => resetBoard()} />

<div class="grid w-[min(100%,60vh)] grid-cols-9 grid-rows-9">
	{#each tiles as tile, i}
		<button
			class="aspect-square w-full"
			onclick={() => tileClicked(i)}
			oncontextmenu={(e) => handleContextMenu(e, i)}
		>
			<Tile value={tile.value} visible={tile.visible} flagged={tile.flagged} />
		</button>
	{/each}
	<div>
		<button onclick={resetBoard} class="m-4 btn preset-filled">Reset</button>
	</div>
</div>
