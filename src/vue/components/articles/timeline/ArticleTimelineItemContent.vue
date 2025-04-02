<template>
    <div class="timeline-item-content">
        <div class="timeline-item-content-header">
            <div class="timeline-item-content-header-title">
                <h4 class="fw-bold"
                    v-html="title"/>

                <InlineInfoList :items="infoListItems"/>
            </div>

            <div class="timeline-item-content-header-date-badge-wrapper">
                <InfoBadge  class="date-badge"
                            fa-icon="fa-regular fa-calendar"
                            :value="dateString"/>
            </div>
        </div>

        <!-- Bouton pour afficher/masquer les références -->
        <button v-if="referenceDetails" class="btn btn-outline-primary mt-2" @click="toggleReferences">
            {{ showReferences ? 'Masquer le référence' : 'Voir référence' }}
        </button>

        <!-- Bloc Références -->
        <div v-if="showReferences && referenceDetails" class="card mt-2">
            <div class="card-body">
                <strong>Référence :</strong> <br>
                <template v-if="referenceDetails.name">
                    <i class="fa-solid fa-user"></i> {{ referenceDetails.name }} <br>
                </template>
                <template v-if="referenceDetails.job">
                    <i class="fas fa-briefcase"></i> {{ referenceDetails.job }} <br>
                </template>
                <template v-if="reference.email">
                    <i class="fa-solid fa-envelope"></i> <a :href="'mailto:' + referenceDetails.email"> {{ referenceDetails.email }}</a> <br>
                </template>
                <template v-if="reference.phone">
                    <i class="fa-solid fa-phone"></i> <a :href="'tel:' + referenceDetails.phone"> {{ referenceDetails.phone }}</a>
                </template>
            </div>
        </div>

        <div class="timeline-item-content-body">
            <p class="text-4 text-default description m-0"
               v-html="description"/>

            <div class="tags-wrapper mt-2 pt-1 mt-md-3">
                <Tags v-if="parsedTags && parsedTags.length"
                      :tags="parsedTags"/>
            </div>
        </div>
    </div>
</template>

<script setup>
import {computed, inject, ref} from "vue"
import InlineInfoList from "/src/vue/components/widgets/InlineInfoList.vue"
import InfoBadge from "/src/vue/components/widgets/InfoBadge.vue"
import Tags from "/src/vue/components/widgets/Tags.vue"

const props = defineProps({
    title: String,
    formattedDateStart: String,
    formattedDateEnd: String,
    province: String,
    country: String,
    institution: String,
    description: String,
    tags: Object|String,
    reference: {
        type: [Object, null],
        default: null,
        validator: (value) => {
            return value === null || (
                typeof value === 'object' && 
                (value.name || value.email || value.phone)
            );
        }
    }
})

/** @type {{value: Boolean}} */
const isScreenXlOrLarger = inject("isScreenXlOrLarger")
const showReferences = ref(false);

const toggleReferences = () => {
    showReferences.value = !showReferences.value;
};

const referenceDetails = computed(() => {

    if (!props.reference) return null;
    
    // Si c'est un objet valide
    if (typeof props.reference === 'object' && (props.reference.name || props.reference.email)) {
        return props.reference;
    }
    
    // Si c'est une chaîne "locales.reference", on la filtre
    if (typeof props.reference === 'string' && props.reference.includes('locales.')) {
        return null;
    }
    
    return props.reference;
});

const parsedTags = computed(() => {
    if(Array.isArray(props.tags))
        return props.tags
    return []
})

const dateString = computed(() => {
    return props.formattedDateStart +
        '<i class="mx-2 fa-solid fa-arrow-right-long" style="font-size: 10px; opacity: 0.85"></i>' +
        props.formattedDateEnd
})

const locationString = computed(() => {
    if(!props.province && !props.country)
        return null

    return `
        ${props.province ?? ''}
        ${props.province && props.country ? '-' : ''}
        ${props.country ?? ''}
    `
})

const institutionString = computed(() => {
    if(isScreenXlOrLarger.value)
        return props.institution

    const shortenedLocation = props.province || props.country

    if(!props.institution && shortenedLocation)
        return locationString.value
    if(!shortenedLocation && props.institution)
        return props.institution

    return `${props.institution}
        <span class="mx-1 opacity-50">·</span>
        ${shortenedLocation}
    `
})

const infoListItems = computed(() => {
    const compress = !isScreenXlOrLarger.value

    return [
        {
            faIcon: `fa-regular fa-clock`,
            value: dateString.value,
            visibility: ['mobile']
        },

        {
            faIcon: `fa-regular fa-building`,
            value: institutionString.value,
            visibility: ['desktop', 'mobile']
        },

        {
            faIcon: `fa-regular fa-font-awesome`,
            value: locationString.value,
            visibility: ['desktop']
        }
    ].filter(item => item.value).filter(item => item.visibility.includes(compress ? 'mobile' : 'desktop'))
})
</script>

<style lang="scss" scoped>
@import "/src/scss/_theming.scss";

div.timeline-item-content {
    flex-grow: 1;
    margin-left: 20px;
    @include media-breakpoint-down(md) {
        margin-left: 16px;
    }
    @include media-breakpoint-down(sm) {
        margin-left: 12px;
    }
}

div.timeline-item-content-header {
    display: flex;
    padding-top: 1rem;
    width: 100%;
    @include media-breakpoint-down(md) {
        padding-top: 0.7rem;
    }
    @include media-breakpoint-down(sm) {
        padding-top: 0.4rem;
    }

    &-title {
        flex-grow: 1;
    }

    @include media-breakpoint-down($navigation-sidebar-breakpoint) {
        flex-direction: column;
    }
}

.date-badge {
    margin-left: 30px;
    min-width: 185px;

    @include media-breakpoint-down(xl) {
        display: none!important;
    }
}

div.timeline-item-content-body {
    margin-top: 20px;
    @include media-breakpoint-down(xl) {
        margin-top: 10px;
    }
    @include media-breakpoint-down(md) {
        margin-top: 5px;
    }
}
</style>