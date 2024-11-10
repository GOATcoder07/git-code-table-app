<h1> Table with Search, Grouping, Sorting, and Selectable Salary Aggregation </h1>

<script>
    import employeesData from './employees.json'; // Import JSON file
    let employees = employeesData.employees; // Access the employees array
    let searchTerm = ''; // Store the search query
    let groupBy = ''; // Default is no grouping
    let sortBy = ''; // Store the sort criteria
    let salaryStat = 'none'; // Default salary statistic is 'None'

    // Keep the original order of employees
    let originalEmployees = [...employees];

    // Filter employees based on searchTerm
    $: filteredEmployees = originalEmployees.filter(employee => {
        return employee.some(field =>
            field.toString().toLowerCase().includes(searchTerm.toLowerCase())
        );
    });

    // Sort employees based on sortBy criteria
    function sortEmployees(a, b) {
        if (sortBy === 'name') {
            return a[0].localeCompare(b[0]);
        } else if (sortBy === 'position') {
            return a[1].localeCompare(b[1]);
        } else if (sortBy === 'office') {
            return a[2].localeCompare(b[2]);
        } else if (sortBy === 'salary') {
            return parseFloat(a[5].replace(/[\$,]/g, '')) - parseFloat(b[5].replace(/[\$,]/g, ''));
        }
        return 0;
    }

    // Compute sorted employees based on sortBy criteria
    $: sortedEmployees = sortBy ? [...filteredEmployees].sort(sortEmployees) : [...filteredEmployees];

    // Function to calculate aggregate, mean, min, max, and median salary
    function calculateSalaryStats(salaries) {
        const total = salaries.reduce((acc, salary) => acc + salary, 0);
        const mean = total / salaries.length;
        const min = Math.min(...salaries);
        const max = Math.max(...salaries);

        // Calculate median
        salaries.sort((a, b) => a - b);
        const mid = Math.floor(salaries.length / 2);
        const median = salaries.length % 2 === 0 ? (salaries[mid - 1] + salaries[mid]) / 2 : salaries[mid];

        return { total, mean, min, max, median };
    }

    // Group employees and calculate aggregated salary statistics
    function groupEmployees() {
        const grouped = {};

        sortedEmployees.forEach(employee => {
            const key = groupBy === 'office' ? employee[2] : employee[1];
            const salary = parseFloat(employee[5].replace(/[\$,]/g, ''));

            if (!grouped[key]) {
                grouped[key] = { employees: [], salaries: [] };
            }
            grouped[key].employees.push(employee);
            grouped[key].salaries.push(salary);
        });

        // Calculate salary stats for each group
        Object.keys(grouped).forEach(key => {
            grouped[key].stats = calculateSalaryStats(grouped[key].salaries);
        });

        return grouped;
    }

    // Regroup employees when groupBy changes
    $: groupedEmployees = groupBy ? groupEmployees() : { 'All Employees': { employees: sortedEmployees, stats: calculateSalaryStats(sortedEmployees.map(e => parseFloat(e[5].replace(/[\$,]/g, '')))) } };
</script>

<!-- Input field for search -->
<input
    type="text"
    placeholder="Search employees..."
    bind:value={searchTerm} />

<!-- Dropdown to select how to group the employees -->
<label for="group-by">Group by: </label>
<select id="group-by" bind:value={groupBy}>
    <option value="">Select</option>
    <option value="office">Office</option>
    <option value="position">Position</option>
</select>

<!-- Dropdown to select how to sort the employees -->
<label for="sort-by">Sort by: </label>
<select id="sort-by" bind:value={sortBy}>
    <option value="">None</option>
    <option value="name">Name</option>
    <option value="position">Position</option>
    <option value="office">Office</option>
    <option value="salary">Salary</option>
</select>

<!-- Dropdown to select salary statistic to display -->
<label for="salary-stat">Show Salary Stat: </label>
<select id="salary-stat" bind:value={salaryStat}>
    <option value="none">None</option>
    <option value="total">Total</option>
    <option value="mean">Mean</option>
    <option value="min">Min</option>
    <option value="max">Max</option>
    <option value="median">Median</option>
</select>

<!-- Display the grouped and filtered employees with selected salary statistic -->
<table>
    <thead>
        <tr>
            <th>Name</th>
            {#if groupBy !== 'office'}
                <th>Office</th>
            {/if}
            {#if groupBy !== 'position'}
                <th>Position</th>
            {/if}
            <th>Extension</th>
            <th>Start Date</th>
            <th>Salary</th>
        </tr>
    </thead>
    <tbody>
        {#if !groupBy}
            {#each sortedEmployees as employee}
                <tr>
                    <td>{employee[0]}</td>
                    <td>{employee[2]}</td>
                    <td>{employee[1]}</td>
                    <td>{employee[3]}</td>
                    <td>{employee[4]}</td>
                    <td>{employee[5]}</td>
                </tr>
            {/each}
            <!-- Conditionally show selected salary stat for all employees -->
            {#if salaryStat !== 'none'}
                <tr>
                    <td colspan="5" style="text-align: right;"><strong>Overall {salaryStat.charAt(0).toUpperCase() + salaryStat.slice(1)} Salary:</strong></td>
                    <td><strong>{groupedEmployees['All Employees'].stats[salaryStat].toLocaleString('en-US', { style: 'currency', currency: 'USD' })}</strong></td>
                </tr>
            {/if}
        {:else}
            {#each Object.keys(groupedEmployees) as key}
                <tr>
                    <td colspan="6">
                        <strong>{groupBy === 'office' ? `Office: ${key}` : `Position: ${key}`}</strong>
                    </td>
                </tr>
                {#each groupedEmployees[key].employees as employee}
                    <tr>
                        <td>{employee[0]}</td>
                        {#if groupBy === 'office'}
                        {:else}
                            <td>{employee[2]}</td>
                        {/if}
                        {#if groupBy === 'position'}
                        {:else}
                            <td>{employee[1]}</td>
                        {/if}
                        <td>{employee[3]}</td>
                        <td>{employee[4]}</td>
                        <td>{employee[5]}</td>
                    </tr>
                {/each}
                <!-- Conditionally show selected salary stat for each group -->
                {#if salaryStat !== 'none'}
                    <tr>
                        <td colspan="4" style="text-align: right;"><strong>{salaryStat.charAt(0).toUpperCase() + salaryStat.slice(1)} Salary:</strong></td>
                        <td><strong>{groupedEmployees[key].stats[salaryStat].toLocaleString('en-US', { style: 'currency', currency: 'USD' })}</strong></td>
                    </tr>
                {/if}
            {/each}
        {/if}
    </tbody>
</table>

<style>
    table {
        width: 100%;
        border-collapse: collapse;
    }
    th, td {
        border: 1px solid #ddd;
        padding: 8px;
        text-align: left;
    }
    th {
        background-color: #f2f2f2;
    }
</style>
            
