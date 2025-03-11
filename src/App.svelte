<script>
	import { slide } from "svelte/transition";
	import cardTemplate from "./cardTemplate.json";
	import columnTemplate from "./columnTemplate.json";

	let bannerStatus = $state(true);
	let contentSection;
	let sectionHeights = {
		top: "10vh",
		banner: "15vh",
		content: "65vh",
		bottom: "10vh",
	};

	let cards = $state([]);
	let numOfCards = $derived(cards.length);
	let selectedCard = $state(null);
	let columnsList = $state([]);
  	let newColumnObj = $state({...columnTemplate});

	function handleBannerStatusClick() {
		bannerStatus = !bannerStatus;
		if (bannerStatus) {
			sectionHeights.banner = "15vh";
			sectionHeights.content = "65vh";
		} else {
			sectionHeights.banner = "0vh";
			sectionHeights.content = "80vh";
		}
	}

  	function showCardDetails(card) {
    	newColumnObj = {...columnTemplate};
		selectedCard = card;
		columnsList = card.columnDetails;
	}

	function addCard() {
    	newColumnObj = {...columnTemplate};
		cardTemplate.cardId = String(cards.length + 1);
		selectedCard = {...cardTemplate};
    	columnsList = [];
	}

	function updateCard(cardId) {
		if (newColumnObj.columnName && newColumnObj.attributeName) {
			newColumnObj.id = columnsList.length + 1;
			columnsList.push({...newColumnObj});
		}

		selectedCard.columnDetails = columnsList;
		if (cardId === cards.length + 1) {
			cards.push(selectedCard);
		} else {
			cards[cardId - 1] = selectedCard;
		}
    	newColumnObj = {...columnTemplate};
	}

	function deleteCard() {
		const input = confirm("Do you really want to delete this card?");
		if (input) {
			let idx = cards.find(item => item.cardId === selectedCard.cardId);
			if (idx) {
				cards.splice(idx, 1);
				selectedCard = null;
			}
		}
	}

	function deleteColumn(id) {
		const index = columnsList.find(item => item.id === id);
		let columnId = index.id;
		if (index) {
			columnsList.splice(columnId - 1, 1);
			if (columnsList.length >= columnId) {
				columnsList.map(item => {
					if (item.id > columnId) {
						item.id = columnId;
						columnId++;
					}
					return item;
				})
			}
			selectedCard.columnDetails = columnsList;
		}
	}

	function handleSelectAggregateOn(column) {
		if (column.aggregateOn === 'valueColumns') {
			let ValueColumns = [];
			columnsList.map((columnItem) => {
				if (columnItem.columnType === "value") {
					ValueColumns.push(columnItem.attributeName);
				}
			});
			column.attributeName = ValueColumns;
		} else {
			column.attributeName = "";
		}
	}

	function handleSelectColumnType(column) {
		if (column.columnType === 'value') {
			column.aggregateType = "";
			column.aggregateOn = "";
			column.attributeName = "";
		}
	}

	async function openFile() {
		 try {
            // Open file picker
            const [fileHandle] = await window.showOpenFilePicker({
                types: [{ description: "Cards Configuration File", accept: { "application/json": [".json"] } }]
            });

            // Get file contents
            const file = await fileHandle.getFile();
            const text = await file.text();

			cards = JSON.parse(text);
			selectedCard = cards.length > 0 ? cards[0] : null;
			columnsList = selectedCard?.columnDetails || [];
			newColumnObj = {...columnTemplate};
        } catch (err) {
            console.error("Error reading file:", err);
        }
	}

	async function saveFile() {
		 try {
            const handle = await window.showSaveFilePicker({
                suggestedName: "cards.json",
                types: [{ description: "Cards Configuration File", accept: { "application/json": [".json"] } }]
            });

            const writable = await handle.createWritable();
            await writable.write(JSON.stringify(cards, null, 2));
            await writable.close();
        } catch (err) {
            console.error("Error saving file:", err);
        }

	}
</script>

<main>
	<div class="top" style="height: {sectionHeights.top};">
		<span class="heading">Cards</span>
		<ul>
			<li><button>Home</button></li>
			<li><button onclick={openFile}>Open</button></li>
			<li><button onclick={saveFile}>Save File</button></li>
			<li>
				<button onclick={handleBannerStatusClick}
					>{bannerStatus ? "Hide Summary" : "Show Summary"}</button
				>
			</li>
		</ul>
	</div>

	{#if bannerStatus}
		<div
			class="banner"
			style="height: {sectionHeights.banner};"
			transition:slide={{ duration: 500 }}
		>
			<div style="flex: 1; display: flex; justify-content:space-around">
				<span style="font-weight: bold;">Cards: {numOfCards}</span>
				<span style="font-weight: bold;">Entitites: 0</span>
				<span style="font-weight: bold;">Attributes: 0</span>
			</div>
			<div style="flex: 1; display:flex; justify-content: space-around;">
				<span style="background-color: #00adb5; font-weight: bold;">Columns: {columnsList.length}</span>
				<span style="background-color: #00adb5; font-weight: bold;">Aggregations: 0</span>
			</div>
		</div>
	{/if}

	<!-- Two-Column Layout Without Scrolling -->
	<div
		class="content"
		style="height: {sectionHeights.content};"
		bind:this={contentSection}
	>
		<div class="column column-1">
			<div class="table-container">
				<table>
					<thead>
						<tr>
							<th>Sr No.</th>
							<th>Card Name</th>
							<th>Card Entity</th>
							<th>Columns</th>
						</tr>
					</thead>
					<tbody>
						{#each cards as card}
							<tr
								onclick={() => showCardDetails(card)}
								style={card.cardId === selectedCard?.cardId
									? "background-color: #E0E0E0"
									: ""}
							>
								<td>{card.cardId}</td>
								<td>{card.cardName}</td>
								<td>{card.entity}</td>
								<td>{card.columnDetails.map(item => item.columnName).join(", ")}</td>
							</tr>
						{/each}
					</tbody>
				</table>
			</div>
		</div>
		<div class="column column-2">
			{#if selectedCard}
				<div style="display: flex; justify-content:space-between;">
					<h2>Card Details</h2>
					<div class="buttons">
						<button onclick={addCard}>New Card</button>
						<button onclick={() => updateCard(selectedCard.cardId)}>Save</button
						>
						<button class="delete-btn" onclick={deleteCard}>Delete</button>
					</div>
				</div>

				<div style="display: flex; gap: 1rem; overflow:auto;">
					<div style="flex:1;">
						<div style="display: flex; justify-content:space-between;">
							<span>Card Id:</span>
							<input type="text" readonly bind:value={selectedCard.cardId} />
						</div>

						<div style="display: flex; justify-content:space-between;">
							<span>Card Name:</span>
							<input type="text" bind:value={selectedCard.cardName} />
						</div>

						<div style="display: flex; justify-content:space-between;">
							<span>Card Type:</span>
							<input type="text" bind:value={selectedCard.cardType} />
						</div>

						<div style="display: flex; justify-content:space-between;">
							<span>Entity:</span>
							<input type="text" bind:value={selectedCard.entity} />
						</div>

						<div style="display: flex; justify-content:space-between;">
							<span>Height:</span>
							<input type="text" readonly bind:value={selectedCard.height} />
						</div>

						<div style="display: flex; justify-content:space-between;">
							<span>Width:</span>
							<input type="text" readonly bind:value={selectedCard.width} />
						</div>

						<div style="display: flex; justify-content:space-between;">
							<span>x:</span>
							<input type="text" readonly bind:value={selectedCard.x} />
						</div>

						<div style="display: flex; justify-content:space-between;">
							<span>y:</span>
							<input type="text" readonly bind:value={selectedCard.y} />
						</div>

						{#each selectedCard.columnDetails as column}
							<div style="display: flex; flex-direction:column; position: relative;">
								<div style="background-color: azure; color: black; margin:4px 0px">
									Column {column.id}
								</div>
								<button class="cross-button" onclick={() => {deleteColumn(column.id)}}>x</button>
								<div>
									<div style="display: flex; justify-content:space-between;">
										<span>Column Name:</span>
										<input type="text" bind:value={column.columnName} />
									</div>

									<div style="display: flex; justify-content:space-between;">
										<span>Column Type:</span>
										<select 
											bind:value={column.columnType}
											onchange={() => handleSelectColumnType(column)}>
											<option selected value="value">value</option>
											<option value="aggregate">aggregate</option>
										</select>
									</div>

									{#if column.columnType === "aggregate"}
										<div style="display: flex; justify-content:space-between;">
											<span>Aggregate Type:</span>
											<select bind:value={column.aggregateType}>
												<option selected value="count">COUNT</option>
												<option value="countComplement">COUNT Complement</option
												>
												<option value="sum">SUM</option>
												<option value="sumComplement">SUM Complement</option>
												<option value="min">MIN</option>
												<option value="max">MAX</option>
												<option value="avg">AVG</option>
												<option value="greaterThan">Greater Than</option>
												<option value="LessThan">Less Than</option>
												<option value="range">RANGE</option>
											</select>
										</div>

										<div style="display: flex; justify-content:space-between;">
											<span>Aggregate On:</span>
											<select
												bind:value={column.aggregateOn}
												onchange={() => handleSelectAggregateOn(column)}
											>
												<option value="valueColumns">Value Columns</option>
												<option value="selectAttribute">Select Attribute</option
												>
											</select>
										</div>
									{/if}

									<div style="display: flex; justify-content:space-between;">
										<span>Attribute Name:</span>
										<input
											type="text"
											readonly={column.aggregateOn === "valueColumns"}
											bind:value={column.attributeName}
										/>
									</div>
								</div>
							</div>
						{/each}

						<div style="display: flex; flex-direction:column;">
							<div style="background-color: azure; color: black; margin: 4px 0px;">
								Add New Column
							</div>
							<div>
								<div style="display: flex; justify-content:space-between;">
									<span>Column Name:</span>
									<input type="text" bind:value={newColumnObj.columnName} />
								</div>

								<div style="display: flex; justify-content:space-between;">
									<span>Column Type:</span>
									<select 
										bind:value={newColumnObj.columnType}
										onchange={() => handleSelectColumnType(newColumnObj)}>
										<option selected value="value">value</option>
										<option value="aggregate">aggregate</option>
									</select>
								</div>

								{#if newColumnObj.columnType === "aggregate"}
									<div style="display: flex; justify-content:space-between;">
										<span>Aggregate Type:</span>
										<select bind:value={newColumnObj.aggregateType}>
											<option selected value="count">COUNT</option>
											<option value="countComplement">COUNT Complement</option>
											<option value="sum">SUM</option>
											<option value="sumComplement">SUM Complement</option>
											<option value="min">MIN</option>
											<option value="max">MAX</option>
											<option value="avg">AVG</option>
											<option value="greaterThan">Greater Than</option>
											<option value="LessThan">Less Than</option>
											<option value="range">RANGE</option>
										</select>
									</div>

									<div style="display: flex; justify-content:space-between;">
										<span>Aggregate On:</span>
										<select
											bind:value={newColumnObj.aggregateOn}
											onchange={() => handleSelectAggregateOn(newColumnObj)}
										>
											<option value="valueColumns">Value Columns</option>
											<option value="selectAttribute">Select Attribute</option>
										</select>
									</div>
								{/if}

								<div style="display: flex; justify-content:space-between;">
									<span>Attribute Name:</span>
									<input
										type="text"
										readonly={newColumnObj.aggregateOn === "valueColumns"}
										bind:value={newColumnObj.attributeName}
									/>
								</div>
							</div>
						</div>
					</div>
					<div
						style="flex:1; display:flex; justify-content:center; align-items:center;"
					>
						{#if selectedCard?.columnDetails?.length > 0}
							<div
								style="display: flex; flex-direction: column; width:400px; background-color: white; border: 1px solid grey;"
							>
								<div style="display:flex; justify-content:center; background-color:black;">{selectedCard.cardName}</div>
								<div style="display: flex;">
									{#each selectedCard.columnDetails as column}
										<span style="flex: 1; padding: 4px; color: {column.columnType === 'value' ? '#0074D9' : '#2ECC40'}; text-align:center;">{column.columnName}</span>
									{/each}
								</div>
								<div style="height: 100px; color:black; display:flex; justify-content:center; align-items:center;">Data</div>
							</div>
						{:else}
							<p>Preview of Card</p>
						{/if}
					</div>
				</div>
			{:else}
				<div style="display: flex; justify-content:flex-end">
					<button onclick={addCard}>New Card</button>
				</div>
				<div
					style="display: flex; justify-content:center; align-items:center; height:100%"
				>
					<p>Select a row to see details.</p>
				</div>
			{/if}
		</div>
	</div>

	<div class="bottom" style="height: {sectionHeights.bottom};">
		<div></div>
		<div>
			{#if selectedCard?.columnDetails?.length > 0}
				<div class="column-legend">
					<div>
						<span style="background-color: #0074D9;"></span>
						<p>Value Column</p>
					</div>
					<div>
						<span style="background-color: #2ECC40;"></span>
						<p>Aggregate Column</p>
					</div>
				</div>
			{/if}
		</div>
	</div>
</main>

<style>
	main {
		height: 100vh;
		display: flex;
		flex-direction: column;
		box-sizing: border-box;
		overflow: hidden;
	}

	.top,
	.bottom {
		background-color: #eeeeee;
		flex: 0 0 5vh;
	}

	.top {
		display: flex;
		justify-content: space-between;
		padding: 1rem;
		box-sizing: border-box;
	}

	.banner {
		background-color: #393e46;
		display: flex;
		justify-content: space-around;
		transition: all 1s ease;
		flex: 0 0 15vh; /* Fixed height */
	}

	.content {
		flex: 1;
		display: grid;
		grid-template-columns: 50% 50%;
		gap: 1rem;
		box-sizing: border-box;
		overflow: hidden;
		padding: 1rem;
		width: 100%;
	}

	.column {
		border-radius: 0.5rem;
		display: flex;
		justify-content: center;
		font-weight: bold;
		min-width: 0;
		min-height: 0;
		margin-top: 2rem;
	}

	.column-2 {
		background-color: #00adb5;
		color: white;
		display: flex;
		flex-direction: column;
		justify-content: flex-start;
		padding: 1rem;
	}

	.table-container {
		width: 100%;
		height: 100%;
		border-radius: 8px;
		overflow-y: auto;
		border: 1px solid #222831;
	}

	table {
		width: 100%;
		border-collapse: collapse;
		background: white;
		border-radius: 8px;
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
		table-layout: fixed;
	}

	thead {
		position: sticky;
		top: 0;
		background-color: #222831;
		color: white;
		z-index: 2;
	}

	th {
		background-color: #222831;
		color: white;
		padding: 12px;
		text-align: left;
	}

	tbody tr {
		height: 40px;
		color: black;
	}

	td {
		padding: 0.75rem;
		text-align: left;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}

	tr:hover {
		background-color: rgba(0, 0, 0, 0.05);
		cursor: pointer;
	}

	ul {
		list-style: none;
		display: flex;
		align-items: center;
		gap: 0.5rem;
		padding: 0;
		margin: 0;
	}

	li {
		color: black;
	}

	.banner > div > span {
		width: 20%;
		min-height: 8vh;
		height: 50%;
		align-self: center;
		/* background-color: #00adb5; */
		background-color: #222831;
		border-radius: 0.5rem;
		display: flex;
		justify-content: center;
		align-items: center;
		border-bottom: 2px solid white;
	}

	.heading {
		font-size: 2rem;
		font-weight: 700;
		display: flex;
		align-items: center;
		color: black;
	}

  	input, select {
		width: 40%;
		height: 2rem;
		padding: 4px 6px;
		margin-bottom: 4px;
		font-size: 1rem;
		border: 2px solid #ccc;
		border-radius: 6px;
		outline: none;
		background-color: white;
		color: black;
		transition: border-color 0.3s ease, box-shadow 0.3s ease;
		box-sizing: border-box;
	}

	input:hover, select:hover {
		border-color: #007bff;
	}

	input:focus, select:focus {
		border-color: #007bff;
		box-shadow: 0 0 5px rgba(0, 123, 255, 0.5);
	}

	input[readonly] {
		background-color: #f0f0f0;
		color: #555;
		border-color: #ddd;
		cursor: not-allowed;
	}

	.cross-button {
		padding:0;
		margin:0;
		width: 3rem;
		height: 26px;
		position: absolute;
		right: 0px;
		top: 2px;
		display: flex;
		justify-content: center;
		align-items: center;
		border-radius: 0;
	}

	.bottom {
		display: flex;
	}

	.bottom > div {
		flex: 1;
		display: flex;
		justify-content: flex-end;
	}

	.column-legend {
		width: 25%;
		display: flex;
		justify-content: center;
		align-items: center;
		gap: 1rem;
	}

	.column-legend > div {
		display: flex;
		justify-content: center;
		align-items: center;
	}

	.column-legend > div > span {
		width: 10px;
		height: 10px;
		margin: 4px;
	}

	.column-legend > div > p {
		color: black;
		font-weight: 600;
	}
</style>
