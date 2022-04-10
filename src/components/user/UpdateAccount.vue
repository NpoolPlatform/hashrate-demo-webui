<template>
  <UpdatePage
    @submit='onSubmit'
    :label='accountType === AccountType.Email ? "MSG_UPDATE_EMAIL" : "MSG_UPDATE_MOBILE"'
    submit-text='MSG_SUBMIT'
    v-model:account='oldAccount'
    v-model:account-type='oldAccountType'
    v-model:verification-code='oldVerificationCode'
    v-model:verification-code-error='oldVerificationCodeError'
  >
    <template #top-right>
      <div class='switcher'>
        <q-icon
          class='icon'
          size='1.5em'
          :name='accountType === AccountType.Mobile ? "smartphone" : "email"'
        />
      </div>
    </template>
    <template #body>
      <PhoneNO
        v-if='accountType === AccountType.Mobile'
        v-model:value='phoneNO'
        :error='accountError'
        @focus='onPhoneNOFocusIn'
        @blur='onPhoneNOFocusOut'
      />
      <Input
        v-if='accountType === AccountType.Email'
        v-model:value='emailAddress'
        label='MSG_EMAIL_ADDRESS'
        type='email'
        id='email'
        required
        :error='accountError'
        message='MSG_EMAIL_TIP'
        placeholder='MSG_EMAIL_PLACEHOLDER'
        @focus='onEmailFocusIn'
        @blur='onEmailFocusOut'
      />
      <q-btn class='send-code alt' @click='onSendCodeClick'>
        {{ $t('MSG_SEND_CODE') }}
      </q-btn>
      <Input
        v-model:value='myVerificationCode'
        :label='accountType === AccountType.Email ? "MSG_EMAIL_VERIFICATION_CODE" : "MSG_MOBILE_VERIFICATION_CODE"'
        type='text'
        id='ver-code'
        required
        :error='verificationCodeError'
        message='MSG_VERIFICATION_CODE_TIP'
        placeholder='MSG_VERIFICATION_CODE_PLACEHOLDER'
        @focus='onVerificationCodeFocusIn'
        @blur='onVerificationCodeFocusOut'
      />
    </template>
  </UpdatePage>
</template>

<script setup lang='ts'>
import {
  validateEmailAddress,
  AccountType,
  validateMobileNO,
  validateVerificationCode,
  useCodeRepoStore,
  useLangStore,
  MessageUsedFor,
  NotificationType,
  useUserStore
} from 'npool-cli-v2'
import { defineAsyncComponent, ref, toRef, watch, defineProps } from 'vue'
import { useI18n } from 'vue-i18n'
import { useRouter } from 'vue-router'

interface Props {
  accountType: AccountType
}

const props = defineProps<Props>()
const accountType = toRef(props, 'accountType')

// eslint-disable-next-line @typescript-eslint/unbound-method
const { t } = useI18n({ useScope: 'global' })

const UpdatePage = defineAsyncComponent(() => import('src/components/user/UpdatePage.vue'))
const Input = defineAsyncComponent(() => import('src/components/input/Input.vue'))
const PhoneNO = defineAsyncComponent(() => import('src/components/input/PhoneNO.vue'))

const oldAccount = ref('')
const oldAccountType = ref(AccountType.Email)
const oldVerificationCode = ref('')
const oldVerificationCodeError = ref(false)

const account = ref('')
const accountError = ref(false)

const phoneNO = ref('')
const onPhoneNOFocusIn = () => {
  accountError.value = false
}
const onPhoneNOFocusOut = () => {
  accountError.value = !validateMobileNO(phoneNO.value)
}
watch(phoneNO, () => {
  if (accountType.value === AccountType.Mobile) {
    account.value = phoneNO.value
  }
})

const emailAddress = ref('')
const onEmailFocusIn = () => {
  accountError.value = false
}
const onEmailFocusOut = () => {
  accountError.value = !validateEmailAddress(emailAddress.value)
}
watch(emailAddress, () => {
  if (accountType.value === AccountType.Mobile) {
    account.value = emailAddress.value
  }
})

const myVerificationCode = ref('')
const verificationCodeError = ref(false)
const onVerificationCodeFocusIn = () => {
  verificationCodeError.value = false
}
const onVerificationCodeFocusOut = () => {
  verificationCodeError.value = !validateVerificationCode(myVerificationCode.value)
}

const coderepo = useCodeRepoStore()
const lang = useLangStore()
const user = useUserStore()
const router = useRouter()

const onSubmit = () => {
  if (accountError.value || verificationCodeError.value || oldVerificationCodeError.value) {
    return
  }

  user.updateAccount({
    Account: account.value,
    AccountType: accountType.value,
    VerificationCode: myVerificationCode.value,
    VerificationCodes: [
      {
        Account: oldAccount.value,
        AccountType: oldAccountType.value,
        VerificationCode: oldVerificationCode.value
      }
    ],
    Message: {
      Error: {
        Title: t('MSG_UPDATE_ACCOUNT'),
        Message: t('MSG_UPDATE_ACCOUNT_FAIL'),
        Popup: true,
        Type: NotificationType.Error
      }
    }
  }, () => {
    void router.back()
  })

  return false
}

const onSendCodeClick = () => {
  if (!account.value?.length) {
    return
  }

  switch (accountType.value) {
    case AccountType.Email:
      coderepo.sendEmailCode({
        LangID: lang.CurLang?.ID as string,
        EmailAddress: account.value,
        UsedFor: MessageUsedFor.Update,
        ToUsername: account.value,
        Message: {
          Error: {
            Title: t('MSG_SEND_EMAIL_CODE'),
            Message: t('MSG_SEND_EMAIL_CODE_FAIL'),
            Popup: true,
            Type: NotificationType.Error
          }
        }
      })
      break
    case AccountType.Mobile:
      coderepo.sendSMSCode({
        LangID: lang.CurLang?.ID as string,
        PhoneNO: account.value,
        UsedFor: MessageUsedFor.Signup,
        Message: {
          Error: {
            Title: t('MSG_SEND_SMS_CODE'),
            Message: t('MSG_SEND_SMS_CODE_FAIL'),
            Popup: true,
            Type: NotificationType.Error
          }
        }
      })
      break
  }
}

</script>

<style lang='sass' scoped>
.switcher
  width: 0
  height: 0
  border-width: 32px
  border-style: solid
  border-color: #ff964a #ff964a transparent transparent
  border-top-right-radius: 12px

.icon
  margin-right: -48px
  margin-top: -56px
</style>