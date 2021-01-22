<script>
    import {onDestroy, onMount} from 'svelte'

    import ReadPerMinute from '../core/ReadPerMinute'
    import DOMWaiter from '../core/DOMWaiter'
    import interpolate from '../utils/interpolate'

    export let selector
    export let lang = 'en'
    export let template = '%time% min read'

    let time = 0
    let words = 0
    let error = null

    let waiter = null

    onMount(async () => {
        if (!selector) {
            return
        }
        try {
            waiter = new DOMWaiter()
            const el = await waiter.wait(selector)

            const y = el.getBoundingClientRect().y
            const h = el.getBoundingClientRect().height
            document.addEventListener('scroll', () => {
                console.log(Math.ceil(Math.abs(el.getBoundingClientRect().y - y) / h * 100))
            })
            el.addEventListener('scroll', () => console.log('scroll'))

            const rdm = new ReadPerMinute()
            ;({time, words} = rdm.parse(el.textContent, lang))
        } catch (err) {
            error = err.message
        }
    })

    onDestroy(() => {
        !!waiter && waiter.unwait()
    })
</script>

{#if $$slots.content}
    <slot name="content" {time} {words}/>
{:else}
    <span data-testid='__readotron-root__' {...$$restProps}>
        {#if !!time && !!template}
            {interpolate(template, {time, words}, '%')}
        {/if}
        {#if !!error}
            {error}
        {/if}
    </span>
{/if}
