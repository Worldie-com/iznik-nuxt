<template>
  <div>
    <chat-message-text
      v-if="chatmessage.type === 'Default'"
      :chat="chat"
      :chatmessage="chatmessage"
      :otheruser="otheruser"
      :pov="pov"
      :chatusers="chatusers"
      :highlight-emails="highlightEmails"
    />
    <chat-message-image
      v-else-if="chatmessage.type === 'Image'"
      :chat="chat"
      :chatmessage="chatmessage"
      :otheruser="otheruser"
      :pov="pov"
      :chatusers="chatusers"
    />
    <chat-message-interested
      v-else-if="chatmessage.type === 'Interested' && otheruser"
      :chat="chat"
      :chatmessage="chatmessage"
      :otheruser="otheruser"
      :pov="pov"
      :chatusers="chatusers"
      :highlight-emails="highlightEmails"
    />
    <chat-message-completed
      v-else-if="chatmessage.type === 'Completed' && otheruser"
      :chat="chat"
      :chatmessage="chatmessage"
      :otheruser="otheruser"
      :pov="pov"
      :chatusers="chatusers"
    />
    <chat-message-promised
      v-else-if="chatmessage.type === 'Promised' && otheruser"
      :chat="chat"
      :chatmessage="chatmessage"
      :otheruser="otheruser"
      :pov="pov"
      :chatusers="chatusers"
    />
    <chat-message-reneged
      v-else-if="chatmessage.type === 'Reneged' && otheruser"
      :chat="chat"
      :chatmessage="chatmessage"
      :otheruser="otheruser"
      :pov="pov"
      :chatusers="chatusers"
    />
    <chat-message-address
      v-else-if="chatmessage.type === 'Address' && otheruser"
      :chat="chat"
      :chatmessage="chatmessage"
      :otheruser="otheruser"
      :pov="pov"
      :chatusers="chatusers"
    />
    <chat-message-nudge
      v-else-if="chatmessage.type === 'Nudge' && otheruser"
      :chat="chat"
      :chatmessage="chatmessage"
      :otheruser="otheruser"
      :pov="pov"
      :chatusers="chatusers"
    />
    <chat-message-mod-mail
      v-else-if="chatmessage.type === 'ModMail'"
      :chat="chat"
      :chatmessage="chatmessage"
      :otheruser="null"
      :pov="pov"
      :chatusers="chatusers"
    />
    <chat-message-schedule
      v-else-if="chatmessage.type === 'Schedule' && otheruser"
      :chat="chat"
      :chatmessage="chatmessage"
      :otheruser="otheruser"
      :pov="pov"
      :chatusers="chatusers"
    />
    <chat-message-schedule
      v-else-if="chatmessage.type === 'ScheduleUpdated' && otheruser"
      :chat="chat"
      :chatmessage="chatmessage"
      :otheruser="otheruser"
      :pov="pov"
      :chatusers="chatusers"
    />
    <chat-message-report
      v-else-if="chatmessage.type === 'ReportedUser'"
      :chat="chat"
      :chatmessage="chatmessage"
      :otheruser="otheruser"
      :pov="pov"
      :chatusers="chatusers"
    />
    <div v-else-if="supportOrAdmin">
      Unknown chat message type {{ chatmessage.type }}
    </div>
    <chat-message-warning :chatmessage="chatmessage" />
    <chat-message-date-read
      :chat="chat"
      :chatmessage="chatmessage"
      :otheruser="otheruser"
      :last="last"
      :pov="pov"
      :chatusers="chatusers"
    />
  </div>
</template>
<script>
// Don't use dynamic imports because it stops us being able to scroll to the bottom after render.
import ChatMessageWarning from '@/components/ChatMessageWarning'
import ChatMessageText from './ChatMessageText'
import ChatMessageImage from './ChatMessageImage'
import ChatMessageInterested from './ChatMessageInterested'
import ChatMessageCompleted from './ChatMessageCompleted'
import ChatMessagePromised from './ChatMessagePromised'
import ChatMessageReneged from './ChatMessageReneged'
import ChatMessageAddress from './ChatMessageAddress'
import ChatMessageNudge from './ChatMessageNudge'
import ChatMessageDateRead from './ChatMessageDateRead'
import ChatMessageModMail from './ChatMessageModMail'
import ChatMessageSchedule from './ChatMessageSchedule'
import ChatMessageReport from './ChatMessageReport'

// System chat message doesn't seem to be used; ReportedUser is for ModTools only.

export default {
  components: {
    ChatMessageWarning,
    ChatMessageDateRead,
    ChatMessageText,
    ChatMessageImage,
    ChatMessageInterested,
    ChatMessageCompleted,
    ChatMessagePromised,
    ChatMessageAddress,
    ChatMessageReneged,
    ChatMessageNudge,
    ChatMessageModMail,
    ChatMessageSchedule,
    ChatMessageReport
  },
  props: {
    chat: {
      type: Object,
      required: true
    },
    chatmessage: {
      type: Object,
      required: true
    },
    otheruser: {
      required: true,
      validator: prop => typeof prop === 'object' || prop === null
    },
    last: {
      type: Boolean,
      required: false,
      default: false
    },
    pov: {
      type: Number,
      required: false,
      default: null
    },
    chatusers: {
      type: Array,
      required: true
    },
    highlightEmails: {
      type: Boolean,
      required: false,
      default: false
    }
  }
}
</script>
