<script>
    let count = 0;
    function handleIncrease() {
        count++
    }
</script>

<h1>Svelte is up n' running</h1>
<button on:click={handleIncrease}>Current count is {count}</button>

<style>
button {
    background: blue;
    cursor: pointer;
    color: white;
    font-weight: 700;
    border: none;
    padding: 8px 16px;
    border-radius: 12px;
}
</style>