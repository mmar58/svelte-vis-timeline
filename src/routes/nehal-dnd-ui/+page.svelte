<script>
	import { DataSet, DataView, Queue } from "vis-data";
	import { Timeline } from "vis-timeline";
	import { onMount } from "svelte";
	// import "vis-timeline/styles/vis-timeline-graph2d.css";

	let timeline1;
	let container;

	// Create groups
	const numberOfGroups = 3;
	const groups = new DataSet();
	for (let i = 0; i < numberOfGroups; i++) {
		groups.add({
			id: i,
			content: `Group ${i}`,
		});
	}

	// Create items
	const numberOfItems = 10;
	const items = new DataSet();
	const itemsPerGroup = Math.round(numberOfItems / numberOfGroups);

	for (let truck = 0; truck < numberOfGroups; truck++) {
		let date = new Date();
		for (let order = 0; order < itemsPerGroup; order++) {
			date.setHours(0);
			const start = new Date(date);
			date.setHours(4);
			const end = new Date(date);

			items.add({
				id: order + itemsPerGroup * truck,
				className:'bg-slate-800',
				group: truck,
				start: new Date().setHours(0, 0, 0, 0),
				end: end,
				content: `already added ${order}`,
			});
		}
	}
	let todayStart = new Date().setHours(0, 0, 0, 0);
	let todayEnd = new Date().setHours(0, 0, 30, 0);

	const options = {
		stack: true,
		start: todayStart, // Start at midnight today
		end: todayEnd, // End at the last minute of today
		min: todayStart,
		max: new Date().setHours(20, 0, 0, 0),
		zoomMin: 1000 * 60 * 1, // Minimum zoom to 1 hour
		zoomMax: 1000 * 60 * 60 * 60, // Maximum zoom to 12 hours
		editable: true,
		orientation: "top",
		onDropObjectOnItem: (objectData, item, callback) => {
			if (!item) return;
			alert(
				`dropped object with content: "${objectData.content}" to item: "${item.content}"`,
			);
		},
		moveable: true,
		zoomable: true,
		horizontalScroll: true,
		// stack: false, // if items overlap, try disabling stacking
		orientation: { axis: "top", item: "top" },
		multiselect: true,
		// Customize the time axis to show only hours
		format: {
			minorLabels: minorLabels,
			majorLabels: majorLabels,
		},
		// editable: { updateTime: true, updateGroup: true }
	};
	function minorLabels(date, scale, step) {
		switch (scale) {
			case "millisecond":
				return date.format("HH:mm:ss");
			case "second":
				return date.format("HH:mm:ss");
			case "minute":
				return date.format("HH:mm");
			case "hour":
				return date.format("HH:mm");
			case "weekday":
				return date.format("ddd D MMMM");
			case "day":
				return date.format("ddd D MMMM");
			case "week":
				return date.format("ddd D MMMM");
			case "month":
				return date.format("MMMM YYYY");
			case "year":
				return date.format("YYYY");
		}
	}
	function majorLabels(date, scale, step) {
		switch (scale) {
			case "millisecond":
				return date.format("HH:mm:ss");
			case "second":
				return date.format("HH:mm:ss");
			case "minute":
				return date.format("HH:mm");
			case "hour":
				return date.format("HH:mm");
			case "weekday":
				return date.format("ddd D MMMM");
			case "day":
				return date.format("ddd D MMMM");
			case "week":
				return date.format("ddd D MMMM");
			case "month":
				return date.format("MMMM YYYY");
			case "year":
				return date.format("YYYY");
		}
	}
	const handleDragStart = (event) => {
		console.log(" handle drag item",event)
		event.dataTransfer.effectAllowed = "move";
		// const itemType = event.target.innerHTML.split("-")[1].trim();
		const itemType = "range"
		const item = {
			id: new Date(),
			type: itemType,
			content: event.target.innerHTML.split("-")[0].trim(),
		};

		const isFixedTimes =
			event.target.innerHTML.split("-")[2] &&
			event.target.innerHTML.split("-")[2].trim() === "fixed times";

		if (isFixedTimes) {
			item.start = new Date();
			item.end = new Date(1000 * 60 * 10 + new Date().valueOf());
		}

		event.dataTransfer.setData("text", JSON.stringify(item));
	};

	const handleObjectItemDragStart = (event) => {
		event.dataTransfer.effectAllowed = "move";
		const objectItem = {
			content: "objectItemData",
			target: "item",
		};
		event.dataTransfer.setData("text", JSON.stringify(objectItem));
	};

	onMount(() => {
		// Initialize timeline
		timeline1 = new Timeline(container, items, groups, options);

		// Add drag event listeners
		const itemElements = document.querySelectorAll(".items .item");
		const objectItems = document.querySelectorAll(".object-item");

		itemElements.forEach((item) => {
			item.addEventListener("dragstart", handleDragStart);
		});

		objectItems.forEach((objectItem) => {
			objectItem.addEventListener("dragstart", handleObjectItemDragStart);
		});

		// Cleanup function
		return () => {
			if (timeline1) {
				timeline1.destroy();
			}
		};
	});
</script>

<main>
	<h1>Timeline Drag & Drop Example</h1>

	<p>
		For this to work, you will have to define your own
		<code>'dragstart'</code> eventListener on each item in your list of
		items (make sure that any new item added to the list is attached to this
		eventListener 'dragstart' handler). This 'dragstart' handler must set
		<code>dataTransfer</code> - notice you can set the item's information as
		you want this way.
	</p>

	<div bind:this={container} id="visualization"></div>

	<div class="items-panel">
		<div class="side">
			<h3>Items:</h3>
			<ul class="items">
				<li draggable="true" class="item">item 1 - range</li>
				<li draggable="true" class="item">item 2 - point</li>
				<li draggable="true" class="item">item 3 - range</li>
				 
			</ul>
		</div>

		<div class="side">
			<h3>Object with "target:'item'":</h3>
			<li draggable="true" class="object-item">
				object with target:'item'
			</li>
		</div>
	</div>
</main>

<style>
	/* Add your CSS styles here */
	.items-panel {
		display: flex;
		gap: 2rem;
	}

	.side {
		flex: 1;
	}

	.items {
		list-style-type: none;
		padding: 0;
	}

	.item,
	.object-item {
		cursor: move;
		margin-bottom: 0.5rem;
		padding: 0.5rem;
		background: #f5f5f5;
		border: 1px solid #ddd;
		border-radius: 4px;
	}

	li.item {
		list-style: none;
		width: 150px;
		color: #1a1a1a;
		background-color: #d5ddf6;
		border: 1px solid #97b0f8;
		border-radius: 2px;
		margin-bottom: 5px;
		padding: 5px 12px;
	}
	li.item:before {
		content: "≣";
		font-family: Arial, sans-serif;
		display: inline-block;
		font-size: inherit;
		cursor: move;
	}
	li.object-item {
		list-style: none;
		width: 150px;
		color: #1a1a1a;
		background-color: #d5ddf6;
		border: 1px solid #97b0f8;
		border-radius: 2px;
		margin-bottom: 5px;
		padding: 5px 12px;
	}
	li.object-item:before {
		content: "≣";
		font-family: Arial, sans-serif;
		display: inline-block;
		font-size: inherit;
		cursor: move;
	}
	.items-panel {
		display: flex;
		justify-content: space-around;
	}
</style>
