<template>
    <v-app>
        <v-container>
            <Level />
        </v-container>
    </v-app>
</template>

<script>
import Level from './components/Level';

export default {
    name: 'App',
    components: {
        Level,
    }
};
</script>
