<template>
  <div>
    <vuci-form :uci-config="config" :key="rerenderKey">
      <vuci-typed-section :type="section" :columns="columns">
        <template #interface_name="{ s }" name="interface_name">
          <vuci-form-item-dummy :uci-section="s" name="interface_name" />
        </template>
        <template #address="{ s }">
          <vuci-form-item-dummy :uci-section="s" name="address" />
        </template>
        <template #netmask="{ s }">
          <vuci-form-item-dummy :uci-section="s" name="netmask" />
        </template>
        <template #actions="{ s }">
          <a-button type="primary" :uci-section="s" @click="edit(s['.name'])">{{ $t('Edit') }}</a-button>
          <a-popconfirm title="Are you sure delete this task?" ok-text="Yes" cancel-text="No"
            @confirm="remove(s['.name'])" >
            <a-button class="button-margin" :uci-section="s" type="danger"> {{ $t('Delete') }} </a-button>
          </a-popconfirm>
        </template>
      </vuci-typed-section>
      <label name="intname" for="intname">{{ $t('Interface name') }}</label>
      <a-input class="input-width" id="intname" name="intname" v-model="interface_name" placeholder="Interface name" />
      <a-button class="button-right" :disabled="!interface_name" type="primary" @click="handleAdd">Create</a-button>
    </vuci-form>
    <a-modal v-model="isVisible" :title="modalTitle" :mask-closable="false">
      <template #footer></template>
      <ModalWindow :isVisible="isVisible" :editableField="editableField" />
    </a-modal>
  </div>
</template>

<script>
import ModalWindow from './interfaces/ModalWindow.vue'
export default {
  components: {
    ModalWindow
  },

  data () {
    return {
      config: 'vuci_components_task',
      section: 'interface',
      columns: [
        { name: 'interface_name', label: 'Interface name' },
        { name: 'address', label: 'IP Address' },
        { name: 'netmask', label: 'Netmask' },
        { name: 'actions' }
      ],
      isVisible: false,
      editableField: '',
      rerenderKey: 1,
      interface_name: '',
      sections: [],
      editable_interface_name: ''
    }
  },

  methods: {
    async load () {
      await this.$uci.load(this.config)
      this.sections = this.$uci.sections(this.config, this.section)
      this.rerenderKey++
    },
    modalTitle () {
      return `Edit ${this.editable_interface_name}`
    },
    edit (sid) {
      this.editable_interface_name = this.sections
        .filter(section => section['.name'] === sid)
        .map(section => section.interface_name)
      this.editableField = sid
      this.isVisible = !this.isVisible
      this.load()
    },
    remove (sid) {
      this.$uci.del(this.config, sid)
      this.$uci.save().then(() => {
        this.$uci.apply().then(() => this.load())
      })
    },
    handleAdd () {
      if (!this.interface_name || this.interface_name.length === 0) {
        return this.$message.warning(this.$t('Interface name can not be empty...'))
      }
      const validationSameName = this.sections.some(one => one.interface_name === this.interface_name)
      if (validationSameName) {
        return this.$message.warning(this.$t('Interface already exists'))
      }

      this.add()
    },
    async add () {
      const sid = this.$uci.add(this.config, this.section)
      this.$uci.set(this.config, sid, 'interface_name', this.interface_name)

      await this.$uci.save()
      await this.$uci.apply()

      await this.load()
      const last = this.getLastSection()
      if (last) {
        this.edit(last['.name'])
      }
    },
    getLastSection () {
      return this.sections[this.sections.length - 1]
    },
    checkAndEdit () {
      const checker = this.getLastSection()
      if ((!checker.address || !checker.netmask) && checker.proto !== 'dhcp') {
        this.edit(checker['.name'])
      }
    }
  },
  async created () {
    await this.load()
    this.checkAndEdit()
  },
  watch: {
    isVisible () {
      this.load()
    }
  }
}
</script>
<style>
.button-margin {
  margin-left: 10px
}
.input-width {
  margin-right: 20px;
  margin-left: 20px;
  width: 30%;
}
.button-right {
  float: right;
  margin-right: 100px;
}
</style>
