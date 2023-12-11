<template>
  <div>
    <vuci-form v-if="isVisible" :uci-config="config">
      <vuci-named-section :name="editableField" v-slot="{ s }">
        <vuci-form-item-select :uci-section="s" :label="'Protocol'" name="proto" :options="proto" initial="static"
          required />
      <vuci-form-item-input :uci-section="s" :label="'IP address'" name="address" depend="proto == 'static'" rules="ipaddr" required/>
      <vuci-form-item-input :uci-section="s" :label="'Netmask'" name="netmask" depend="proto == 'static'" rules="netmask4" required/>
      <vuci-form-item-input :uci-section="s" :label="'Gateway'" name="gateway" depend="proto == 'static'" rules="ipaddr"/>
      <vuci-form-item-list :uci-section="s" :label="'DNS'" name="dns" depend="proto == 'static'" />
      </vuci-named-section>
    </vuci-form>
  </div>
</template>

<script>
export default {
  props: {
    isVisible: {
      type: Boolean,
      default: false
    },
    editableField: {
      type: String,
      default: ''
    }
  },

  data () {
    return {
      config: 'vuci_components_task',
      section: 'interface',
      proto: ['dhcp', 'static']
    }
  }

}
</script>
