<!--
  - SPDX-FileCopyrightText: 2018 Nextcloud contributors
  - SPDX-License-Identifier: AGPL-3.0-or-later
-->

<template>
	<div class="poll-info-line">
		<span v-for="(subText) in subTexts" :key="subText.id" :class="['sub-text', subText.class]">
			<Component :is="subText.iconComponent" :size="16" />
			<span class="sub-text">{{ subText.text }}</span>
		</span>
	</div>
</template>

<script>
import { mapState, mapGetters } from 'vuex'
import moment from '@nextcloud/moment'
import unpublishedIcon from 'vue-material-design-icons/PublishOff.vue'
import archivedPollIcon from 'vue-material-design-icons/Archive.vue'
import closedPollIcon from 'vue-material-design-icons/Lock.vue'
import creationIcon from 'vue-material-design-icons/ClockOutline.vue'
import ProposalsIcon from 'vue-material-design-icons/Offer.vue'
import ExpirationIcon from 'vue-material-design-icons/CalendarEnd.vue'

export default {
	name: 'PollInfoLine',

	computed: {
		...mapState({
			pollConfiguration: (state) => state.poll.configuration,
			pollStatus: (state) => state.poll.status,
			ownerDisplayName: (state) => state.poll.owner.displayName,
			mayEdit: (state) => state.poll.permissions.edit,
		}),

		...mapGetters({
			isPollClosed: 'poll/isClosed',
			hasShares: 'shares/hasShares',
			isProposalExpirySet: 'poll/isProposalExpirySet',
			isProposalExpired: 'poll/isProposalExpired',
			proposalsExpireRelative: 'poll/proposalsExpireRelative',
		}),

		isNoAccessSet() {
			return this.pollConfiguration.access === 'private' && !this.hasShares && this.mayEdit
		},

		subTexts() {
			const subTexts = []

			if (this.pollStatus.deleted) {
				subTexts.push({
					id: 'deleted',
					text: t('polls', 'Archived'),
					class: 'archived',
					iconComponent: archivedPollIcon,
				})
				return subTexts
			}

			if (this.isNoAccessSet) {
				subTexts.push({
					id: 'no-access',
					text: [
						t('polls', 'Unpublished'),
					].join('. '),
					class: 'unpublished',
					iconComponent: unpublishedIcon,
				})
				return subTexts
			}

			if (this.pollConfiguration.access === 'private') {
				subTexts.push({
					id: this.pollConfiguration.access,
					text: t('polls', 'A private poll from {name}', { name: this.ownerDisplayName }),
					class: '',
					iconComponent: null,
				})
			} else {
				subTexts.push({
					id: this.pollConfiguration.access,
					text: t('polls', 'An openly accessible poll from {name}', { name: this.ownerDisplayName }),
					class: '',
					iconComponent: null,
				})
			}

			if (this.isPollClosed) {
				subTexts.push({
					id: 'closed',
					text: this.timeExpirationRelative,
					class: 'closed',
					iconComponent: closedPollIcon,
				})
				return subTexts
			}

			if (!this.isPollClosed && this.pollConfiguration.expire) {
				subTexts.push({
					id: 'expiring',
					text: t('polls', 'Closing {relativeExpirationTime}', { relativeExpirationTime: this.timeExpirationRelative }),
					class: this.closeToClosing ? 'closing' : 'open',
					iconComponent: ExpirationIcon,
				})
				return subTexts
			}

			if (this.isProposalExpirySet && this.isProposalExpired) {
				subTexts.push({
					id: 'expired',
					text: t('polls', 'Proposal period ended {timeRelative}', { timeRelative: this.proposalsExpireRelative }),
					class: 'proposal',
					iconComponent: ProposalsIcon,
				})
				return subTexts
			}

			if (this.isProposalExpirySet && !this.isProposalExpired) {
				subTexts.push({
					id: 'proposal-open',
					text: t('polls', 'Proposal period ends {timeRelative}', { timeRelative: this.proposalsExpireRelative }),
					class: 'proposal',
					iconComponent: ProposalsIcon,
				})
				return subTexts
			}

			if (subTexts.length < 2) {
				subTexts.push({
					id: 'created',
					text: this.dateCreatedRelative,
					class: 'created',
					iconComponent: creationIcon,
				})
			}
			return subTexts
		},

		dateCreatedRelative() {
			return moment.unix(this.pollStatus.created).fromNow()
		},

		closeToClosing() {
			return (!this.isPollClosed && this.pollConfiguration.expire && moment.unix(this.pollConfiguration.expire).diff() < 86400000)
		},

		timeExpirationRelative() {
			if (this.pollConfiguration.expire) {
				return moment.unix(this.pollConfiguration.expire).fromNow()
			}
			return t('polls', 'never')

		},
	},
}

</script>

<style lang="scss">
.poll-info-line {
	display: flex;
	flex-wrap: wrap;
	opacity: 0.7;
	font-size: 1em;

	.material-design-icon {
		padding: 0 2px;
	}

	.sub-text {
		display: flex;
	}

	& > span:not(:last-child)::after {
		content: "|";
		padding: 0 2px;
	}

	.closed, .archived {
		.sub-text{
			color: var(--color-error);
			font-weight: 700;
		}
	}

	.unpublished, .open {
		.sub-text{
			font-weight: 700;
		}
	}

	.closing {
		.sub-text{
			color: var(--color-warning);
			font-weight: 700;
		}
	}

	.created {
		.sub-text{
			color: var(--color-main-text);
		}
	}
}
</style>
