<script setup lang="ts">
import { computed, ref } from 'vue';
import { useRouter } from 'vue-router';
import {
  mdiAccountOutline,
  mdiArrowLeft,
  mdiTrashCanOutline,
  mdiEmailOutline,
  mdiLogout,
} from '@mdi/js';

import { appRoutes } from '@/main';
import { useUserStore } from '@/stores/userStore';

import BackgroundComponent from '@/components/design-system/style/BackgroundComponent.vue';
import ButtonComponent from '@/components/design-system/input/ButtonComponent.vue';
import BodyText from '@/components/design-system/typography/BodyText.vue';
import CardComponent from '@/components/design-system/containers/CardComponent.vue';
import FormComponent from '@/components/design-system/input/FormComponent.vue';
import FooterMultiWord from '@/components/FooterMultiWord.vue';
import HeadlineText from '@/components/design-system/typography/HeadlineText.vue';
import IconComponent from '@/components/design-system/icons/IconComponent.vue';
import IconButton from '@/components/design-system/input/IconButton.vue';
import ModalComponent from '@/components/design-system/other/ModalComponent.vue';
import NavigationBar from '@/components/design-system/navigation/NavigationBar.vue';
import ProfileCardComponent from './ProfileCardComponent.vue';
import ResponsiveWrapper from '@/components/utilities/ResponsiveWrapper.vue';
import SecondaryNavigationBar from '@/components/design-system/navigation/SecondaryNavigationBar.vue';
import StackComponent from '@/components/design-system/layout/StackComponent.vue';
import StaticContentLayout from '@/components/layouts/StaticContentLayout.vue';
import StickyHeaderLayout from '@/components/layouts/StickyHeaderLayout.vue';
import TextInputComponent from '@/components/design-system/input/TextInputComponent.vue';
import TitleText from '@/components/design-system/typography/TitleText.vue';

const router = useRouter();
const userStore = useUserStore();
const user = ref(userStore.user);
const formSavedSuccess = ref();
const saving = ref();

const handlePreviousClick = () => router.go(-1);

const isModalOpen = ref(false);
const handleOpenModal = () => (isModalOpen.value = true);
const handleCloseModal = () => (isModalOpen.value = false);

const handleDeleteUserClick = async () => {
  const response = await fetch('/api/users/me', { method: 'DELETE' });

  if (response.ok) {
    isModalOpen.value = false;
    userStore.fetchUser();
    router.push(appRoutes.index);
  } else {
    throw Error('User removal failed');
  }
};

const onSubmit = async () => {
  formSavedSuccess.value = '';
  saving.value = true;

  const response = await fetch('/api/users/me', {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(user.value),
  });

  if (response.ok) {
    formSavedSuccess.value = 'Vaše změny byly úspěšne zapsány';
    saving.value = false;
    userStore.fetchUser();
  } else {
    saving.value = false;
    throw Error('User update failed');
  }
};

const validateInput = (value: string | undefined) => {
  const length = value?.length;

  if (typeof length === 'number') return length < 1 ? 'Vyplňte pole' : null;
};

const handleLogout = async () => {
  const response = await fetch('/api/auth/logout');
  if (response.ok) {
    userStore.fetchUser();
    router.push(appRoutes.index);
  } else {
    throw Error('Logout failed');
  }
};

if (!userStore.user) router.push(appRoutes.login);
</script>

<template>
  <BackgroundComponent>
    <StickyHeaderLayout>
      <template #header>
        <NavigationBar transparent />
      </template>

      <template #sticky-header>
        <SecondaryNavigationBar centered-title transparent padding="small">
          <template #before>
            <IconButton @click="handlePreviousClick">
              <IconComponent :icon="mdiArrowLeft" title="Zpátky" />
            </IconButton>
          </template>
          <TitleText tag="p" size="medium">Nastavení profilu</TitleText>
        </SecondaryNavigationBar>
      </template>

      <StaticContentLayout size="small" class="content">
        <StackComponent spacing="large" centered>
          <FormComponent
            class="w-full"
            @submit.prevent="onSubmit"
            :success="formSavedSuccess"
          >
            <TextInputComponent
              label="Celé jméno"
              type="text"
              placeholder="Jan Novák"
              class="w-full"
              :modelValue="user?.displayName"
              :icon="mdiAccountOutline"
              @update:modelValue="
                (newVal) => {
                  if (user) user.displayName = newVal;
                }
              "
              :error="validateInput(user?.displayName)"
            />

            <TextInputComponent
              disabled
              required
              label="E-mail"
              type="email"
              placeholder="E-mail"
              class="w-full"
              :modelValue="user?.email"
              :icon="mdiEmailOutline"
              @update:modelValue="
                (newVal) => {
                  if (user) user.email = newVal;
                }
              "
            />

            <ButtonComponent
              kind="filled"
              color="primary"
              class="w-full"
              :loading="saving"
            >
              Uložit změny
            </ButtonComponent>
          </FormComponent>

          <img src="/images/lock_person.svg" width="32" height="32" alt="" />

          <StackComponent centered spacing="extra-small">
            <BodyText size="small">
              <strong>Jak nakládáme s daty?</strong>
            </BodyText>

            <BodyText size="small">
              Přectěte si o tom na <a href=".">Zásady ochrany dat</a>
            </BodyText>
          </StackComponent>
          <StackComponent centered spacing="extra-small" class="w-full">
            <ButtonComponent
              kind="filled"
              color="white"
              class="w-full"
              @click.prevent="handleOpenModal"
            >
              Odstránit profil
              <template #iconAfter>
                <IconComponent :icon="mdiTrashCanOutline" />
              </template>
            </ButtonComponent>

            <ButtonComponent
              kind="filled"
              color="white"
              class="w-full"
              @click.prevent="handleLogout"
            >
              Odhlásit se
              <template #iconAfter>
                <IconComponent :icon="mdiLogout" />
              </template>
            </ButtonComponent>
          </StackComponent>
        </StackComponent>
      </StaticContentLayout>
    </StickyHeaderLayout>

    <StaticContentLayout>
      <FooterMultiWord />
    </StaticContentLayout>
  </BackgroundComponent>
  <ModalComponent
    teleport-selector=".theme-provider"
    :close-modal-callback="handleCloseModal"
    :is-modal-open="isModalOpen"
  >
    <template #title>
      <TitleText tag="h2" size="small">Opravdu chcete smazat profil?</TitleText>
    </template>
    <template #content>
      <StackComponent spacing="large" centered>
        <span />

        <img src="/images/lock_person.svg" width="32" height="32" alt="" />

        <BodyText size="medium">
          Smazáním profilu přijdete permanentne o uložené kalkulačky.
        </BodyText>

        <ResponsiveWrapper extra-small small>
          <StackComponent spacing="small" stretched class="w-full">
            <ButtonComponent
              kind="filled"
              size="medium"
              color="primary"
              @click.prevent="handleDeleteUserClick"
              center
            >
              Smazat profil a data
            </ButtonComponent>
            <ButtonComponent
              kind="outlined"
              size="medium"
              color="neutral"
              @click.prevent="handleCloseModal"
            >
              Ponechat profil a data
            </ButtonComponent>
          </StackComponent>
        </ResponsiveWrapper>

        <ResponsiveWrapper medium large extra-large huge>
          <StackComponent horizontal spacing="small" stretched>
            <ButtonComponent
              kind="filled"
              size="medium"
              color="primary"
              @click.prevent="handleDeleteUserClick"
            >
              Smazat profil a data
            </ButtonComponent>
            <ButtonComponent
              kind="outlined"
              size="medium"
              color="neutral"
              @click.prevent="handleCloseModal"
            >
              Ponechat profil a data
            </ButtonComponent>
          </StackComponent>
        </ResponsiveWrapper>
      </StackComponent>
    </template>
  </ModalComponent>
</template>

<style lang="scss" scoped>
.w-full {
  width: 100%;
}

.content {
  padding-top: 5%;
  padding-bottom: 20%;

  @media (min-width: 767px) {
    padding-top: 3%;
    padding-bottom: 6%;
  }
}
</style>
