<script>
	// @ts-nocheck
	// import writable stores 
	import { writable } from 'svelte/store';

	export let data;

	let categories = data.categories;
	let locations = data.locations;

	let filtered = writable([...locations]);

	// Get session (returned from layout)
	let { session } = data;
	$: ({ session } = data);



	/**
	 * @param id {number}
	 */
	async function filterByCat(id = 0) {
		let endpoint = '/api/locations/';

		if (id != 0) {
			endpoint = `${endpoint}category/${id}`;
		}

		console.log('api endpoint: ', endpoint);

		// Call fetch
		const response = await fetch(endpoint);

		// if resonse code 200 (ok)
		if (response.ok) {
			// get json from resonse
			const json = await response.json();
			locations = json.data;
		}
	};

		//  keep track of sort directions for each col
		const table_sort = {
			id: false,
			name: false,
			description: false,
			shared: false,
			favourite: false
		};
		

		// Sort alpha values in a given column
		function sortAlpha(col) {
			// reverse current sort direction for this col
			// i.e. reverse the current order
			table_sort[col] = !table_sort[col];

			// Update sort icon class
			updateIcon(col);

			// output to the  browser console
			console.log(`${col} : ${table_sort[col]}`);

			// sort the products array based on column selected
			// sort takes a function parameter to indicate which column should be sorted
			// For JS ternery operator see:
			// https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_operator
			$filtered = $filtered.sort((a, b) => {
				// sort asc
				if (table_sort[col] === true) {
					return a[col].toLowerCase() < b[col].toLowerCase() ? -1 : 1;
					// sort desc
				} else {
					return a[col].toLowerCase() > b[col].toLowerCase() ? -1 : 1;
				}
			});
		};

		// sort numeric and boolean values in a given column
		function sortNumeric(col) {
			// reverse current sort direction for this col
			table_sort[col] = !table_sort[col];

			// Update sort icon class
			updateIcon(col);

			// check browser console for output
			// sort takes a function parameter to indicate which column should be sorted
			console.log(`${col} : ${table_sort[col]}`);

			// sort the products array based on column selected
			$filtered = $filtered.sort((a, b) => {
				if (table_sort[col] === true) {
					return a[col] - b[col];
				} else {
					return b[col] - a[col];
				}
			});
		};
	
		// Sort alpha values in a given column
		function sortBool(col) {
			// reverse current sort direction for this col
			// i.e. reverse the current order
			table_sort[col] = !table_sort[col];

			// Update sort icon class
			updateIcon(col);

			// output to the  browser console
			console.log(`${col} : ${table_sort[col]}`);

			// sort the products array based on column selected
			// sort takes a function parameter to indicate which column should be sorted
			// For JS ternery operator see:
			// https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_operator
			$filtered = $filtered.sort((a, b) => {
				// sort asc
				if (table_sort[col] === true) {
					return a[col] - b[col] ? -1 : 1;
					// sort desc
				} else {
					return a[col] - b[col] ? -1 : 1;
				}
			});
		};

	async function delete_loc(id = 0) {
		if (confirm(`Permanently deleting product with ID= ${id}\n\nAre you sure?`)) {
			// call the store function if user confirms
			const result = await fetch(`/api/locations/${id}`, {
				method: 'DELETE'
			});

			// show the result
			alert('delete result: ', result);
			console.log('delete: ', result);
			
			filterByCat();
		}
	}

	function updateIcon(col){
		const sortIcon = document.getElementById(col);
		sortIcon.classList.toggle('bi-sort-down');
		sortIcon.classList.toggle('bi-sort-up');
	}
</script>

<!-- The HTML content of the page-->

<div class="container">
	<div class="row">
		<!-- Page Header -->
		<h2 class="mt-5">Locations from Supabase</h2>
	</div>
	{#if categories && locations}
		<div class="row">
			<div class="col-sm-1">
				<!-- Page Body Left Column (menu) -->
				<div id="categories" class="list-group">
					<button on:click={() => filterByCat(0)} class="list-group-item list-group-item-action">
						All Locations
					</button>
					{#each categories as cat}
						<button
							on:click={() => filterByCat(Number(cat.id))}
							class="list-group-item list-group-item-action"
						>
							{cat.name}
						</button>
					{/each}
				</div>
			</div>
			<!-- End category col -->

			<div class="col-sm-11">
				<!-- Page Body Right Side (Content goes here) -->
				<div id="locations">
					<table class="table table-striped table-bordered table-hover">
						<thead>
							<tr>
								<th on:click={() => sortNumeric('id')}><i class ="bi bi-sort-down" id='id'></i>id</th>
								<th on:click={() => sortAlpha('name')}><i class ="bi bi-sort-down" id='name'></i>Name</th>
								<th on:click={() => sortAlpha('description')}><i class ="bi bi-sort-down" id='description'></i>Description</th>
								<th>Location (lat, long)</th>
								<th on:click={() => sortBool('shared')}><i class ="bi bi-sort-down" id='shared'></i>Shared</th>
								<th on:click={() => sortBool('favourite')}><i class ="bi bi-sort-down" id='favourite'></i>Favourite</th>
							</tr>
						</thead>
						<tbody>
							{#each $filtered as location}
								<tr>
									<td>{location.id}</td>
									<td>{location.name}</td>
									<td>{location.description}</td>
									<td
										><a
											href="https://www.openstreetmap.org/search?query={location.latitude}%2C{location.longitude}#map=17/{location.latitude}/{location.longitude}"
											target="_blank">{location.latitude}, {location.longitude}</a
										>
									</td>
									<td>{location.shared}</td>
									<td>{location.favourite}</td>
									<!-- check if logged in user is owner-->
									<!-- ? makes user.id optional in case it is null-->
									{#if session?.user.id == location.user_id && location.user_id != null}
									<td><a
											class="btn btn-sm btn-outline-primary"
											href="/add_up_location/{location.id}"
											role="button">
											<span class="bi bi-pencil-square" />
										</a>
									</td>
									<td>
										<button
											on:click={() => delete_loc(location.id)}
											class="btn btn-sm btn-outline-danger">
											<span class="bi bi-trash" />
										</button>
									</td>
									{/if}
								</tr>
							{/each}
						</tbody>
					</table>
				</div>
			</div>
			<!-- End locations col -->
		</div>
		<!-- End Row -->
	{:else}
		<p>No data to display</p>
	{/if}
	<p><a href="/add_up_location">Add new location</a></p>
</div>
<!-- End Main Content-->
