<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated class="transparent-header">
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="toggleLeftDrawer"
        />
        <q-toolbar-title>Космос в цифрах и графиках</q-toolbar-title>
        <div>SpaceApp v0.1</div>
      </q-toolbar>
    </q-header>

    <q-drawer v-model="leftDrawerOpen" show-if-above bordered>
      <q-list>
        <q-item-label header>Важные разделы</q-item-label>

        <EssentialLink
          v-for="link in essentialLinks"
          :key="link.title"
          v-bind="link"
        />
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent, ref } from 'vue';
import EssentialLink from 'components/EssentialLink.vue';

const linksList = [
  {
    title: 'Успешные запуски',
    caption: 'quasar.dev',
    icon: 'rocket',
    link: 'https://quasar.dev',
  },
  {
    title: 'Миссии',
    caption: 'github.com/quasarframework',
    icon: 'book',
    link: 'https://github.com/quasarframework',
  },
  {
    title: 'Ракеты',
    caption: 'chat.quasar.dev',
    icon: 'rocket',
    link: 'https://chat.quasar.dev',
  },
  {
    title: 'Компании и страны',
    caption: 'forum.quasar.dev',
    icon: 'flag',
    link: 'https://forum.quasar.dev',
  },
];

export default defineComponent({
  name: 'MainLayout',

  components: {
    EssentialLink,
  },

  setup() {
    const leftDrawerOpen = ref(false);

    return {
      essentialLinks: linksList,
      leftDrawerOpen,
      toggleLeftDrawer() {
        leftDrawerOpen.value = !leftDrawerOpen.value;
      },
    };
  },
});
</script>
<style>
.transparent-header {
  background: transparent; /* Убирает фон */
  border-bottom: 1px solid rgba(255, 255, 255, 0.3); /* Граница снизу */
  box-shadow: none; /* Убирает тень, если есть */
}
</style>
