<template>
  <div class="flex flex-col gap-10">
    <div class="flex flex-col gap-3">
      <span>Setting your profile for</span>
      <CollectionDropCreatedBy :address="substrateAddress" />
    </div>
    <form class="flex flex-col gap-10" @submit.prevent>
      <!-- name -->
      <NeoField :label="`Your Name`" required :error="!form.name">
        <NeoInput
          v-model="form.name"
          data-testid="create-profile-input-name"
          required
          :placeholder="'Enter Your Name'" />
      </NeoField>

      <!-- bio -->
      <NeoField :label="`Short Bio`" required :error="!form.description">
        <NeoInput
          v-model="form.description"
          data-testid="create-profile-input-bio"
          required
          type="textarea"
          :maxlength="200"
          has-counter
          counter-class="mt-3"
          :placeholder="'introduce yourself in a few words'" />
      </NeoField>

      <!-- profile picture -->
      <NeoField :label="`Upload profile picture`" required :error="!form.image">
        <div class="max-w-full grow">
          <p class="text-k-grey text-sm mb-5">
            Recommended: 400x400px, up to 2MB (JPG, PNG)
          </p>
          <SelectImageField
            v-model="form.image"
            :preview="userProfile?.image" />
        </div>
      </NeoField>

      <!-- banner picture -->
      <NeoField :label="`Upload Cover Image`" required :error="!form.banner">
        <div class="max-w-full grow">
          <p class="text-k-grey text-sm mb-5">
            Recommended: 1440x360px (4:1 aspect ratio), up to 10MB (JPG, PNG)
          </p>
          <SelectImageField
            v-model="form.banner"
            :preview="userProfile?.banner" />
        </div>
      </NeoField>

      <!-- socials -->
      <div>
        <p class="font-bold text-xl mb-4">Link socials</p>
        <div class="flex flex-col gap-4">
          <NeoField
            v-for="social in socialLinks"
            :key="social.name"
            class="my-0">
            <div
              class="w-10 h-10 bg-neutral-3 dark:bg-neutral-11 flex justify-center items-center border-y border-l border-k-grey-fix">
              <component :is="social.icon" />
            </div>
            <NeoInput
              v-model="form[social.model]"
              class="!h-10"
              expanded
              :data-testid="social.testId"
              :placeholder="social.placeholder" />
          </NeoField>
        </div>
      </div>
    </form>
    <NeoButton
      :disabled="submitDisabled"
      variant="k-accent"
      label="Finish customization"
      data-testid="create-profile-submit-button"
      @click="emit('submit', form)" />
  </div>
</template>

<script setup lang="ts">
import { formatAddress } from '@/utils/account'
import { NeoButton, NeoField, NeoIcon, NeoInput } from '@kodadot1/brick'
import { ProfileFormData } from '.'
import SelectImageField from '../SelectImageField.vue'
const { accountId } = useAuth()
const { hasProfile, userProfile } = useProfile()

const substrateAddress = computed(() => formatAddress(accountId.value, 42))

const FarcasterIcon = defineAsyncComponent(
  () => import('@/assets/icons/farcaster-icon.svg?component'),
)

const submitDisabled = computed(
  () => !form.name || !form.description || !form.image || !form.banner,
)

const emit = defineEmits<{
  (e: 'submit', value: ProfileFormData): void
}>()

// form state
const form = reactive<ProfileFormData>({
  address: substrateAddress.value,
  name: '',
  description: '',
  image: null,
  banner: null,
  farcasterHandle: undefined,
  twitterHandle: undefined,
  website: undefined,
})

const socialLinks = [
  {
    name: 'farcaster',
    icon: FarcasterIcon,
    model: 'farcasterHandle',
    placeholder: 'Farcaster Handle',
    testId: 'create-profile-input-farcaster-handle',
  },
  {
    name: 'website',
    icon: () => h(NeoIcon, { icon: 'globe', pack: 'fas' }),
    model: 'website',
    placeholder: 'Website',
    testId: 'create-profile-input-website',
  },
  {
    name: 'twitter',
    icon: () => h(NeoIcon, { icon: 'twitter', pack: 'fab' }),
    model: 'twitterHandle',
    placeholder: 'Twitter Handle',
    testId: 'create-profile-input-twitter-handle',
  },
]

watch(hasProfile, (hasProfile) => {
  if (hasProfile) {
    form.name = userProfile.value?.name ?? ''
    form.description = userProfile.value?.description ?? ''
    form.farcasterHandle = userProfile.value?.socials.find(
      (social) => social.platform === 'Farcaster',
    )?.handle
    form.twitterHandle = userProfile.value?.socials.find(
      (social) => social.platform === 'Twitter',
    )?.handle
    form.website = userProfile.value?.socials.find(
      (social) => social.platform === 'Website',
    )?.handle
  }
})
</script>
