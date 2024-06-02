<script>
	import Tile from './tile.svelte';
	import { getModalStore } from '@skeletonlabs/skeleton';

	const modalStore = getModalStore();

	const SIZE = 9;
	let bombsPlaced = false;

	let tiles = createEmptyBoard();

	function createEmptyBoard() {
		let data = [];
		for (let i = 0; i < SIZE; i++) {
			for (let j = 0; j < SIZE; j++) {
				data.push({ value: 0, visible: false, flagged: false });
			}
		}
		return data;
	}

	function setupBombs(index) {
		let bombCount = 10;
		while (bombCount > 0) {
			const placementIndex = Math.round(Math.random() * SIZE * SIZE) - 1;
			if (tiles[placementIndex].value === '💣' || placementIndex === index) {
				continue;
			}
			tiles[placementIndex].value = '💣';
			bombCount -= 1;
		}

		updateBombCounts();
	}

	function getTileIndex(row, col) {
		return row * SIZE + col;
	}

	function isBombAt(row, col) {
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

	function getTileBombCount(row, col) {
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

	/**
	 *
	 * @param {number} row
	 * @param {number} col
	 * @param {Set<number>} seen
	 */
	function floodFill(row, col, seen) {
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

	/**
	 *
	 * @param {number} index
	 */
	function tileClicked(index) {
		if (!bombsPlaced) {
			setupBombs(index);
			bombsPlaced = true;
		}
		tiles[index].visible = true;

		const row = Math.floor(index / SIZE);
		const col = index % SIZE;
		const seen = new Set();
		floodFill(row, col, seen);

		const clickedBomb = tiles[index].value === '💣';
		if (clickedBomb) {
			setTimeout(() => {
				// Wait to finish the render
				const modal = {
					type: 'alert',
					// Data
					title: 'Game Over',
					body: 'You lose!',
					buttonTextCancel: 'Reset',
					response: () => resetBoard()
				};
				modalStore.trigger(modal);
			});
		}

		if (gameWon()) {
			setTimeout(() => {
				// Wait to finish the render
				const modal = {
					type: 'alert',
					// Data
					title: 'Winner',
					buttonTextCancel: 'Hurray!',
					response: () => resetBoard()
				};
				modalStore.trigger(modal);
			});
		}
	}

	function gameWon() {
		return tiles.filter((tile) => tile.value !== '💣' && !tile.visible).length === 0;
	}

	function resetBoard() {
		tiles = createEmptyBoard();
		bombsPlaced = false;
	}

	function flagTile(index) {
		const tile = tiles[index];
		tile.flagged = !tile.flagged;
		tiles = tiles; // trigger update
	}
</script>

<div class="flex-col">
	<div class="grid grid-cols-9 grid-rows-9">
		{#each tiles as tile, i}
			<button on:click={() => tileClicked(i)} on:contextmenu|preventDefault={() => flagTile(i)}>
				<Tile value={tile.value} visible={tile.visible} flagged={tile.flagged} />
			</button>
		{/each}
	</div>
	<div class="flex-row">
		<button on:click={resetBoard} class="m-4 btn variant-filled">Reset</button>
	</div>
</div>
