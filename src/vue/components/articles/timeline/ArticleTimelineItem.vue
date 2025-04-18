<template>
    <li class="timeline-item"
        :class="!item ? `timeline-item-trailing` : ``">
        <IconView class="timeline-item-icon"
                  :img="item?.img"
                  :fa-icon="item?.fallbackFaIcon"
                  :background-color="item?.fallbackFaIconColor"
                  :prioritize-image="true"
                  :transparency="!item"/>

        <ArticleTimelineItemContent v-if="item"
                                    :title="localize(item.locales, 'title')"
                                    :formatted-date-start="localizeDate(props.item.dateStart)"
                                    :formatted-date-end="localizeDate(props.item.dateEnd)"
                                    :province="localize(item.locales, 'province', true)"
                                    :country="localize(item.locales, 'country', true)"
                                    :institution="localize(item.locales, 'institution')"
                                    :description="localize(item.locales, 'description')"
                                    :reference="getValidReference(item.locales)"
                                    :tags="localize(props.item.locales, 'tags')"/>
    </li>
</template>

<script setup>
import {inject} from "vue"
import IconView from "/src/vue/components/widgets/IconView.vue"
import ArticleTimelineItemContent from "/src/vue/components/articles/timeline/ArticleTimelineItemContent.vue"

const props = defineProps({
    item: {
        /** @type {ArticleItem} **/
        type: Object,
        required: false
    }
})

/** @type {Function} */
const localize = inject("localize")

/** @type {Function} */
const localizeDate = inject("localizeDate");

const getValidReference = (locales) => {
    if (!locales || !locales._localesHash) {
        // console.warn('Structure des locales invalide');
        return null;
    }

    // Accès direct à la référence française ou anglaise
    const ref = locales._localesHash.fr?.reference || 
                locales._localesHash.en?.reference;

    // console.log('Référence extraite:', ref);

    // Validation de la référence
    if (ref && typeof ref === 'object') {
        return ref;
    }

    // console.warn('Aucune référence valide trouvée');
    return null;
};

</script>

<style lang="scss" scoped>
@import "/src/scss/_theming.scss";

li.timeline-item {
    display: flex;
    min-height: calc($article-timeline-image-size);
    &:not(:last-child) { margin-bottom: 70px; }

    @include media-breakpoint-down(md) {
        min-height: $article-timeline-image-size-md;
        &:not(:last-child) { margin-bottom: 60px; }
    }
    @include media-breakpoint-down(sm) {
        min-height: $article-timeline-image-size-sm;
        &:not(:last-child) { margin-bottom: 40px; }
    }
}

div.timeline-item-icon {
    width: $article-timeline-image-size;
    min-width: $article-timeline-image-size;
    height: $article-timeline-image-size;
    font-size: calc($article-timeline-image-size/3);
    border: $article-timeline-border-width solid $article-timeline-border-color;

    z-index: 1;
    border-radius: 100%;
    overflow: hidden;
    background-color: $article-timeline-line-color;

    @include media-breakpoint-down(md) {
        width: $article-timeline-image-size-md;
        min-width: $article-timeline-image-size-md;
        height: $article-timeline-image-size-md;
        font-size: calc($article-timeline-image-size-md/3);
        border-width: $article-timeline-border-width-md;
    }
    @include media-breakpoint-down(sm) {
        width: $article-timeline-image-size-sm;
        min-width: $article-timeline-image-size-sm;
        height: $article-timeline-image-size-sm;
        font-size: calc($article-timeline-image-size-sm/3);
        border-width: $article-timeline-border-width-sm;
    }
}

li.timeline-item-trailing {
    min-height: 0;
    div.timeline-item-icon {
        @each $breakpoint, $size in (xxxl: $article-timeline-image-size, md: $article-timeline-image-size-md, sm: $article-timeline-image-size-sm) {
            @include media-breakpoint-down($breakpoint) {
                $offsetHalf: calc($size / 2);
                $offsetQuarter: calc($size / 4);

                width: $offsetHalf;
                min-width: $offsetHalf;
                height: $offsetHalf;
                margin-left: $offsetQuarter;
                margin-top: 0;
            }
        }
    }
}
</style>