<script lang="ts">
    import '../app.css';
    let { children } = $props();
</script>

<main>
    {@render children()}
</main>

<style>
    main {
        min-height: 100vh;
        display: grid;
        place-items: center;
        padding: 12px;
        background: linear-gradient(145deg, rgba(59,130,246,0.08), rgba(249,250,251,0.9));
    }
</style>
