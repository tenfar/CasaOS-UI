<!--
  * @LastEditors: zhanghengxin ezreal.zhang@icewhale.org
  * @LastEditTime: 2023/2/16 下午4:15
  * @FilePath: /CasaOS-UI/src/components/forms/Ports.vue
  * @Description:
  *
  * Copyright (c) 2023 by IceWhale, All Rights Reserved.
  -->
<template>
	<div class="mb-5">
		<div class="field is-flex is-align-items-center mb-2">
			<label class="label mb-0 is-flex-grow-1">{{ $t('Ports') }}</label>
			<b-button icon-left="plus" rounded size="is-small" @click="addItem">{{ $t('Add') }}</b-button>
		</div>
		<div v-if="items.length == 0" class="is-flex is-align-items-center mb-5 info">
			<b-icon class="mr-2 " icon="information" size="is-small"></b-icon>
			<span>
		        {{ $t('No ports now, click “+” to add one.') }}
            </span>

		</div>
		<div v-for="(item,index) in items" :key="'port'+index+item.protocol" class="port-item">
			<b-icon class="is-clickable" icon="close" size="is-small" @click.native="removeItem(index)"></b-icon>
			<ValidationObserver ref="ob" v-slot="{ invalid }" slim>
				<template v-if="index < 1">
					<b-field grouped>
						<validation-provider v-if="showHostPost" v-slot="{errors,valid}"
											 :rules="'yaml_port|not_in_ports:'+  invalidPortsInUse(item.published, item.protocol)"
											 slim>
							<b-field :label="$t('Host')"
									 :type="{ 'is-danger': errors[0], 'is-success': valid}"
									 expanded>
								<b-input :placeholder="$t('Host')"
										 :value="item.host_ip?`${item.host_ip}:`:'' + item.published" expanded
										 @blur="(event, val)=> assignPortsItem(event.target._value, item)"
								></b-input>
							</b-field>
						</validation-provider>

						<validation-provider v-slot="{errors,valid}" rules="yaml_port" slim>
							<b-field :label="$t('Container')" :type="{ 'is-danger': errors[0], 'is-success': valid }"
									 expanded>
								<b-input v-model.number="item.target"
										 :placeholder="$t('Container')"
										 expanded
								></b-input>
							</b-field>
						</validation-provider>


						<b-field :label="$t('Protocol')" expanded>
							<b-select v-model="item.protocol" :placeholder="$t('Protocol')" expanded>
								<option value="tcp">TCP</option>
								<option value="udp">UDP</option>
								<option value="both">TCP + UDP</option>
							</b-select>
						</b-field>
					</b-field>

				</template>
				<template v-else>
					<b-field grouped>
						<validation-provider v-slot="{errors,valid}"
											 :rules="'yaml_port|not_in_ports:'+  invalidPortsInUse(item.published, item.protocol)"
											 slim>
							<b-field
								:type="{ 'is-danger': errors[0], 'is-success': valid}"
								expanded>
								<b-input v-if="showHostPost" :placeholder="$t('Host')"
										 :value="item.host_ip?item.host_ip:'' + item.published" expanded
										 @blur="(event)=> assignPortsItem(event.target._value, item)"
								></b-input>
							</b-field>
						</validation-provider>

						<validation-provider v-slot="{errors,valid}" rules="yaml_port" slim>
							<b-field :type="{ 'is-danger': errors[0], 'is-success': valid }"
									 expanded>
								<b-input v-model.number="item.target" :placeholder="$t('Container')" expanded
								></b-input>
							</b-field>
						</validation-provider>

						<b-select v-model="item.protocol" :placeholder="$t('Protocol')" expanded>
							<option value="tcp">TCP</option>
							<option value="udp">UDP</option>
							<option value="both">TCP + UDP</option>
						</b-select>
					</b-field>


				</template>
			</ValidationObserver>
		</div>

	</div>
</template>

<script>
import {ValidationObserver, ValidationProvider} from 'vee-validate';

export default {
	name: 'ports-form',
	components: {
		ValidationProvider,
		ValidationObserver
	},
	data() {
		return {
			isLoading: false,
			min: 0
		}
	},
	model: {
		prop: 'vData',
		event: 'change'
	},
	props: {
		vData: Array,
		showHostPost: Boolean,
		ports_in_use: {
			default: () => {
				return {tcp: [], udp: []}
			},
			type: Object
		},
	},
	computed: {
		items: {
			get() {
				return this.vData;
			},
			set(val) {
				this.validateField(val);
			}
		},
	},
	methods: {
		addItem() {
			let itemObj = {
				target: "",
				published: "",
				host_ip: "",
				protocol: "tcp"
			}
			this.items.push(itemObj)
		},

		removeItem(index) {
			this.items.splice(index, 1)
		},

		validateField(val) {
			this.$refs.ob.checkStep().then(valid => {
				if (valid) {
					this.$emit('change', val);
				}
			});
		},
		assignPortsItem(val, item) {
			const reg = /((^(\d{1,3}\.){3}\d{1,3}):)?(\d{1,5}$)/;
			const partList = val.match(reg);
			item.host_ip = partList?.[2] || '';
			item.published = partList?.[4] || val;
		},

		/*
		* type : tcp/udp
		* */
		invalidPortsInUse(port, type) {
			// The host's port input is String Type.
			// port = port - 0;
			if (type === 'both') {
				return (this.ports_in_use?.["udp"] || []).includes(port) || (this.ports_in_use?.["tcp"] || []).includes(port)
			}
			if (type) {
				return (this.ports_in_use?.[type] || this.ports_in_use?.[type.toUpperCase()] || []).includes(port + "")
			}
			return false;
		},

		invalidPortsInUseRule(type) {
			if (type === 'both') {
				return (this.ports_in_use?.["udp"] || []).concat((this.ports_in_use?.["tcp"] || [])).join(',')
			}
			if (type) {
				return (this.ports_in_use?.[type] || []).join(',')
			}
		},
	},
}
</script>

<style lang="scss">
.info {
	font-size: 0.875rem;
	color: #5a5a5a;
}

.port-item {
	position: relative;

	.icon {
		position: absolute;
		right: -1.25rem;
		bottom: 0.825rem;
	}

	&:not(:last-child) {
		margin-bottom: 0.5rem;
	}

	.field.is-expanded {
		.label {
			text-align: center;
			font-weight: normal;
		}
	}
}
</style>