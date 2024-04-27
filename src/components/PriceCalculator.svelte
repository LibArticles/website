<script lang="ts">
	interface PricingInfo {
		models: Array<{
			yearlyBasePrice100x: number;
			monthlyBasePrice100x: number;
			userCountBrackets: Array<{
				end?: number;
				yearlyPricePerUser100x: number;
				monthlyPricePerUser100x: number;
			}>;
			name: string;
			iconName: string;
			description: string;
			comingSoon?: boolean;
		}>;
	}

	const pricing: PricingInfo = {
		models: [
			{
				name: 'Articleman for Schools',
				iconName: 'school',
				description:
					"Use Articleman to manage student newspapers, literary magazines, video teams, and yearbooks. We'll verify that you're with an educational institution.",
				yearlyBasePrice100x: 50_00,
				monthlyBasePrice100x: 5_00,
				userCountBrackets: [
					{
						end: 10,
						yearlyPricePerUser100x: 0,
						monthlyPricePerUser100x: 0
					},
					{
						yearlyPricePerUser100x: 1_50,
						monthlyPricePerUser100x: 15
					}
				]
			},
			{
				name: 'Articleman for Independent Media',
				iconName: 'diversity_1',
				description: 'Use Articleman to manage productions for companies of almost any size.',
				yearlyBasePrice100x: 150_00,
				monthlyBasePrice100x: 15_00,
				userCountBrackets: [
					{
						end: 3,
						yearlyPricePerUser100x: 0,
						monthlyPricePerUser100x: 0
					},
					{
						end: 150,
						yearlyPricePerUser100x: 10_00,
						monthlyPricePerUser100x: 1_00
					}
				]
			},
			{
				name: 'Large Scale Articleman',
				iconName: 'trending_up',
				description:
					'Use Articleman in large-scale media businesses, with powerful centralized management, organization and integrations.',
				yearlyBasePrice100x: 1000_00,
				monthlyBasePrice100x: 100_00,
				userCountBrackets: [
					{
						yearlyPricePerUser100x: 10_00,
						monthlyPricePerUser100x: 1_00
					}
				]
			}
		]
	};

	$: totalPrice = calculatePrice();

	$: numStaff = 0;

	$: priceFormatted = totalPrice.toFixed(2);

	$: currentPlan = 0;

	$: yearly = true;

	$: maxStaffAllowed = calculateMaxStaff();

	function calculateMaxStaff() {
		if (!currentPlan) {
			currentPlan = 0;
		}
		return Math.max(
			...pricing.models[currentPlan]!.userCountBrackets.map((bracket) => {
				console.log(bracket.end);
				return bracket.end ?? 31415;
			})
		);
	}

	function calculatePrice() {
		if (!currentPlan) {
			currentPlan = 0;
		}
		if (!numStaff) {
			numStaff = 0;
		}
		if (yearly === undefined) {
			yearly = true;
		}
		const plan = pricing.models[currentPlan]!;
		let basePrice = yearly ? plan?.yearlyBasePrice100x : plan?.monthlyBasePrice100x;
		console.log('bp:', basePrice);
		console.log('pl:', plan);
		console.log('yr:', yearly);
		console.log('ns:', numStaff);

		let remainingUsers = numStaff;

		let previousBracketEnd = 0;

		let staffPrice = 0;

		for (const bracket of plan?.userCountBrackets ?? []) {
			const price = yearly ? bracket.yearlyPricePerUser100x : bracket.monthlyPricePerUser100x;
			if (bracket.end) {
				const numberOfPossibleUsers = bracket.end - previousBracketEnd;

				if (remainingUsers >= numberOfPossibleUsers) {
					staffPrice += numberOfPossibleUsers * price;
					remainingUsers -= numberOfPossibleUsers;
				} else {
					staffPrice += remainingUsers * price;
					break;
				}
			} else {
				staffPrice = remainingUsers * price;
			}
		}

		return (basePrice + staffPrice) / 100;
	}
</script>

<span class="text-9xl font-serif font-bold font-soft-100 w-full text-center"
	><span class="text-caramel-500 text-5xl">$</span>{priceFormatted}<button
		class="text-caramel-500 text-5xl"
		on:click={() => {
			yearly = !yearly;
			totalPrice = calculatePrice();
		}}>{yearly ? '/yr' : '/mo'}</button
	>
</span>

<div class="flex flex-col desktop:flex-row">
	<section class="desktop:w-[46rem] flex-col p-2">
		<h1 class="text-4xl font-serif font-bold font-soft-100 mb-4">Plan</h1>
		{#each pricing.models as pricePoint, index}
			<button
				class="text-left p-2 rounded-xl mb-2 w-full transition-colors duration-75 {index ===
				currentPlan
					? 'bg-caramel-700 text-caramel-100'
					: 'bg-caramel-200'}"
				on:click={() => {
					currentPlan = index;
					maxStaffAllowed = calculateMaxStaff();
					if (numStaff > maxStaffAllowed) {
						numStaff = maxStaffAllowed;
					}
					totalPrice = calculatePrice();
				}}
			>
				<p class="text-lg">{pricePoint.name}</p>
			</button>
		{/each}
		<p class="min-h-36 text-sm px-2">{pricing.models[currentPlan]?.description}</p>
	</section>
	<section class="w-full flex-col p-2">
		<h1 class="text-4xl font-serif font-bold font-soft-100">Staff members</h1>
		<input
			bind:value={numStaff}
			on:input={() => (totalPrice = calculatePrice())}
			type="number"
			min="0"
			max={maxStaffAllowed}
		/>
	</section>
	<section class="w-full flex-col p-2">
		<h1 class="text-4xl font-serif font-bold font-soft-100">Addons</h1>
	</section>
</div>
