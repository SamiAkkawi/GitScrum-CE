<script>
import VueMarkdownIt from 'vue-markdown-it'

export default {
  components: { VueMarkdownIt },
  props: {
    currentObject: {
      type: Object,
      required: false,
      default: function() {
        return []
      },
    },
    text: {
      type: String,
      required: false,
      default: ''
    },
    placeholder: {
      type: String,
      required: false,
      default: ''
    },
  },
	data() {
		return {
			edit: false,
			label: '',
			empty: 'Enter some text value',
			options: {
        html: true,
				markdownIt: {
          linkify: true,
          
				},
				linkAttributes: {
					attrs: {
					target: '_blank',
					rel: 'noopener'
					}
				}
				}
		}
  },
	computed: {
		vplaceholder: function(){
			if(this.placeholder==undefined || this.placeholder==''){
				return this.empty
			}else{
				return this.placeholder
			}
		},
		vlabel: function(){
			if(this.text==undefined||this.text==''){
				return this.vplaceholder
			}else{
				return this.label
			}
		}
	},
	mounted: function(){
		this.initText();
	},
	updated: function(){
		var ed = this.$refs.labeledit;
		if(ed!=null){
			ed.focus();
		}
	},
	watch: {
		text: function(value){
			if(value==''||value==undefined){
				this.label = this.vplaceholder
			}
		}
  },
  methods: {
		initText: function(){
			if(this.text==''||this.text==undefined){
				this.label = this.vlabel
			}else{
				this.label = this.text
			}
		},
		onLabelClick: function(){
			this.edit = true;
			this.label = this.text;
		},
		updateTextBlur: function(){
			this.edit = false;
			this.$emit('text-updated-blur', {text: this.label, object: this.currentObject})
		},
		updateTextEnter: function(){
			this.edit = false;
			this.$emit('text-updated-enter', {text: this.label, object: this.currentObject})
		},
		inputHandler(e) {
			if (e.keyCode === 13 && !e.shiftKey) {
				e.preventDefault();
				this.updateTextEnter(e);
			}
		},
	},
}
</script>
<template>
	<div class="vlabeledit">
    <div @click="onLabelClick">
      <VueMarkdownIt id="" :source="vlabel" :options="options" class="vlabeledit-label" />
    </div>
		<div v-if="edit">
      <b-form-textarea
      ref="labeledit"
      v-model="label"
      :placeholder="vplaceholder"
      rows="3"
      max-rows="6"
      class="vlabeledit-input" 
      @blur="updateTextBlur" 
      @keyup.enter="inputHandler"></b-form-textarea>
      <small class="mt-2 font-weight-bold">
        <font-awesome-icon :icon="['far', 'info-square']" class="mr-1" /> {{ $t('You can you use markdown in description') }}</small>
    </div>
	</div>
</template>
