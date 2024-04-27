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
			staffCountFlavorText?: string;
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
				staffCountFlavorText: '10 staff members included at no cost',
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
				staffCountFlavorText: '3 staff members included at no cost',
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
						yearlyPricePerUser100x: 15_00,
						monthlyPricePerUser100x: 1_50
					}
				]
			}
			// {
			// 	name: 'Large Scale Articleman',
			// 	iconName: 'trending_up',
			// 	description:
			// 		'Use Articleman in large-scale media businesses, with powerful centralized management, organization and integrations.',
			// 	yearlyBasePrice100x: 2000_00,
			// 	monthlyBasePrice100x: 200_00,
			// 	userCountBrackets: [
			// 		{
			// 			yearlyPricePerUser100x: 10_00,
			// 			monthlyPricePerUser100x: 1_00
			// 		}
			// 	]
			// }
		]
	};

	$: totalPrice = calculatePrice();

	$: numStaff = 0;

	$: priceFormatted = totalPrice.toFixed(2);

	$: currentPlan = 0;

	$: yearly = false;

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
			yearly = false;
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

<div class="min-h-screen w-full flex flex-row">
	<div class="min-h-screen w-96 bg-caramel-950 text-caramel-50 p-4">
		<div class="flex flex-col gap-4">
			<h1 class="text-4xl font-serif font-bold font-soft-100">Pricing</h1>
			<section class="flex flex-col gap-4" role="listbox">
				<h2 class="text-3xl font-serif font-bold font-soft-100">Plan</h2>
				{#each pricing.models as pricePoint, index}
					<button
						role="option"
						aria-selected={currentPlan === index}
						class="text-left p-4 rounded-xl w-full transition-colors duration-75 font-bold flex flex-row gap-4 {index ===
						currentPlan
							? 'bg-caramel-100 text-caramel-900'
							: 'bg-caramel-900'}"
						on:click={() => {
							currentPlan = index;
							maxStaffAllowed = calculateMaxStaff();
							if (numStaff > maxStaffAllowed) {
								numStaff = maxStaffAllowed;
							}
							totalPrice = calculatePrice();
						}}
					>
						<span class="material-symbols-rounded">{pricePoint.iconName}</span>
						<p class="text-base">{pricePoint.name}</p>
					</button>
				{/each}
				<p aria-live="polite" class="min-h-24 text-sm px-4 font-bold text-caramel-350">
					{pricing.models[currentPlan]?.description}
				</p>
			</section>
			<section class="flex flex-col gap-4" role="listbox">
				<h2 class="text-3xl font-serif font-bold font-soft-100">Billing</h2>
				<button
					role="option"
					aria-selected={yearly}
					class="text-left p-4 rounded-xl w-full transition-colors duration-75 font-bold flex flex-row gap-4 {!yearly
						? 'bg-caramel-100 text-caramel-900'
						: 'bg-caramel-900'}"
					on:click={() => {
						yearly = false;
						totalPrice = calculatePrice();
					}}
				>
					<span class="material-symbols-rounded">calendar_month</span>
					<p class="text-base">Monthly</p>
				</button>
				<button
					role="option"
					aria-selected={yearly}
					class="text-left p-4 rounded-xl w-full transition-colors duration-75 font-bold flex flex-row gap-4 {yearly
						? 'bg-caramel-100 text-caramel-900'
						: 'bg-caramel-900'}"
					on:click={() => {
						yearly = true;
						totalPrice = calculatePrice();
					}}
				>
					<span class="material-symbols-rounded">autorenew</span>
					<p class="text-base">Yearly</p>
				</button>
			</section>
			<section class="w-full flex flex-col gap-4">
				<h2 class="text-3xl font-serif font-bold font-soft-100">Staff members</h2>
				<input
					bind:value={numStaff}
					on:input={() => (totalPrice = calculatePrice())}
					type="number"
					min="0"
					max={maxStaffAllowed}
					class="bg-caramel-900 p-4 font-bold text-7xl rounded-xl w-full"
				/>
				{#if pricing.models[currentPlan]?.staffCountFlavorText}
					<span aria-live="polite" class="text-caramel-350 text-sm font-bold px-4"
						>{pricing.models[currentPlan]?.staffCountFlavorText}</span
					>
				{/if}
			</section>
			<!-- <section class="w-full flex-col">
				<h2 class="text-3xl font-serif font-bold font-soft-100">Addons</h2>
			</section> -->
		</div>
	</div>

	<div class="h-full p-4">
		<span aria-live="polite" class="text-9xl font-serif font-bold font-soft-100 w-full text-center"
			><span class="text-caramel-500 text-5xl">$</span>{priceFormatted}<span
				class="text-caramel-500 text-5xl">{yearly ? '/yr' : '/mo'}</span
			>
		</span>
	</div>
</div>
