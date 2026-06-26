<script lang="ts">
	import { slide } from 'svelte/transition';

	type AssetClass = 'Cash' | 'Fixed Income' | 'Equity';
	interface Security {
		id: string;
		ticker: string;
		name: string;
		assetClass: AssetClass;
		shares: number;
		price: number;
	}
	const data: Security[] = [
		{
			id: '1',
			ticker: 'USD',
			name: 'US Dollar',
			assetClass: 'Cash',
			shares: 25000,
			price: 1.0
		},
		{
			id: '2',
			ticker: 'EUR',
			name: 'Euro',
			assetClass: 'Cash',
			shares: 10000,
			price: 1.08
		},
		{
			id: '3',
			ticker: 'US10Y',
			name: 'US Treasury 10-Year',
			assetClass: 'Fixed Income',
			shares: 500,
			price: 98.5
		},
		{
			id: '4',
			ticker: 'LQD',
			name: 'iShares iBoxx $ Inv Grade Corp Bond ETF',
			assetClass: 'Fixed Income',
			shares: 1200,
			price: 105.2
		},
		{
			id: '5',
			ticker: 'AAPL',
			name: 'Apple Inc.',
			assetClass: 'Equity',
			shares: 1500,
			price: 185.0
		},
		{
			id: '6',
			ticker: 'MSFT',
			name: 'Microsoft Corporation',
			assetClass: 'Equity',
			shares: 800,
			price: 420.5
		},
		{
			id: '7',
			ticker: 'GOOGL',
			name: 'Alphabet Inc.',
			assetClass: 'Equity',
			shares: 1200,
			price: 165.25
		}
	];

	let collapsed = $state<Record<string, boolean>>({
		Cash: true,
		'Fixed Income': true,
		Equity: true
	});

	const toggleCollapse = (catName: string) => {
		collapsed[catName] = !collapsed[catName];
	};

	// Helper to format currency
	const formatCurrency = (value: number) => {
		return new Intl.NumberFormat('en-US', {
			style: 'currency',
			currency: 'USD',
			minimumFractionDigits: 2,
			maximumFractionDigits: 2
		}).format(value);
	};

	const formatNumber = (value: number) => {
		return new Intl.NumberFormat('en-US', {
			minimumFractionDigits: 0,
			maximumFractionDigits: 2
		}).format(value);
	};

	const formatPercent = (value: number) => {
		return new Intl.NumberFormat('en-US', {
			style: 'percent',
			minimumFractionDigits: 2,
			maximumFractionDigits: 2
		}).format(value);
	};

	// Process data to add calculated fields
	let processedData = $derived(
		data.map((item) => ({
			...item,
			marketValue: item.shares * item.price
		}))
	);

	// Calculate total portfolio value
	let totalValue = $derived(processedData.reduce((sum, item) => sum + item.marketValue, 0));

	// Group by asset class
	let groupedData = $derived(() => {
		const categories: Record<
			AssetClass,
			{ items: typeof processedData; totalMarketValue: number; weight: number }
		> = {
			Cash: { items: [], totalMarketValue: 0, weight: 0 },
			'Fixed Income': { items: [], totalMarketValue: 0, weight: 0 },
			Equity: { items: [], totalMarketValue: 0, weight: 0 }
		};

		processedData.forEach((item) => {
			categories[item.assetClass].items.push(item);
			categories[item.assetClass].totalMarketValue += item.marketValue;
		});

		// Calculate weights
		if (totalValue > 0) {
			(Object.keys(categories) as AssetClass[]).forEach((key) => {
				categories[key].weight = categories[key].totalMarketValue / totalValue;
			});
		}

		return [
			{
				name: 'Cash',
				data: categories['Cash'],
				colorClass: 'bg-[var(--color-cat-cash)] hover:brightness-95'
			},
			{
				name: 'Fixed Income',
				data: categories['Fixed Income'],
				colorClass: 'bg-[var(--color-cat-fixed-income)] hover:brightness-95'
			},
			{
				name: 'Equity',
				data: categories['Equity'],
				colorClass: 'bg-[var(--color-cat-equity)] hover:brightness-95'
			}
		].filter((cat) => cat.data.items.length > 0);
	});

	const categories = $derived(groupedData());
</script>

<div class="mx-auto my-8 max-w-5xl overflow-hidden rounded-lg border border-slate-200 shadow-sm">
	<!-- Table Header -->
	<div
		class="grid grid-cols-[1.5fr_4fr_1.5fr_1.5fr_2fr_1.5fr] gap-2 border-b border-slate-200 px-6 py-4 text-xs font-semibold tracking-wider text-slate-500 uppercase"
	>
		<div>Ticker</div>
		<div>Name</div>
		<div class="text-right">Shares</div>
		<div class="text-right">Price</div>
		<div class="text-right">Market Value</div>
		<div class="text-right">% Weight</div>
	</div>

	<!-- Table Body -->
	<!-- eslint-disable-next-line svelte/require-each-key -->
	{#each categories as category}
		<!-- Category Header Row -->
		<button
			class="grid w-full cursor-pointer grid-cols-[1.5fr_4fr_1.5fr_1.5fr_2fr_1.5fr] gap-2 border-b border-slate-200 px-6 py-3.5 focus:ring-2 focus:ring-slate-400 focus:outline-none focus:ring-inset {category.colorClass}"
			onclick={() => toggleCollapse(category.name)}
		>
			<div class="col-span-4 flex items-center gap-2 font-semibold text-slate-900">
				<svg
					class="h-4 w-4 text-slate-600 transition-transform duration-200 {collapsed[category.name]
						? '-rotate-90'
						: ''}"
					fill="none"
					viewBox="0 0 24 24"
					stroke="currentColor"
				>
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M19 9l-7 7-7-7"
					/>
				</svg>
				{category.name}
			</div>
			<div class="text-right font-semibold text-slate-900">
				{formatCurrency(category.data.totalMarketValue)}
			</div>
			<div class="text-right font-semibold text-slate-900">
				{formatPercent(category.data.weight)}
			</div>
		</button>

		<!-- Collapsible Rows -->
		{#if !collapsed[category.name]}
			<div transition:slide={{ duration: 300 }} class="overflow-hidden border-b border-slate-200">
				<!-- eslint-disable-next-line svelte/require-each-key -->
				{#each category.data.items as item}
					<div
						class="grid grid-cols-[1.5fr_4fr_1.5fr_1.5fr_2fr_1.5fr] items-center gap-2 border-b border-slate-100 px-6 py-3 transition-colors last:border-b-0 hover:bg-slate-50"
					>
						<div>
							<span
								class="rounded border border-slate-200 bg-white px-1.5 py-0.5 font-mono text-[13px] text-slate-600 shadow-sm"
							>
								{item.ticker}
							</span>
						</div>
						<div class="truncate pr-4 text-slate-900">{item.name}</div>
						<div class="text-right text-slate-700">{formatNumber(item.shares)}</div>
						<div class="text-right text-slate-700">{formatCurrency(item.price)}</div>
						<div class="text-right text-slate-900">{formatCurrency(item.marketValue)}</div>
						<div class="text-right text-slate-700">
							{formatPercent(item.marketValue / totalValue)}
						</div>
					</div>
				{/each}
			</div>
		{/if}
	{/each}

	<!-- Table Footer -->
	<div class="grid grid-cols-[1.5fr_4fr_1.5fr_1.5fr_2fr_1.5fr] gap-2 px-6 py-5">
		<div class="col-span-4 text-right font-bold text-slate-900">Total Portfolio</div>
		<div class="text-right font-bold text-slate-900">
			{formatCurrency(totalValue)}
		</div>
		<div class="text-right font-bold text-slate-900">{totalValue > 0 ? '100.00%' : '0.00%'}</div>
	</div>
</div>
