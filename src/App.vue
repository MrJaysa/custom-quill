<template>
  <div class="grid">
    <div>
      <div class="mb-5 flex">
        <div class="mx-1">
          <label for="fname">Select First name</label>
          <select id="fname" v-model="fname" @change="setMaker">
            <option value="Joel">Joel</option>
            <option value="Sammy">Sammy</option>
            <option value="James">James</option>
          </select>
        </div>
        <div class="mx-1">
          <label for="lname">Select Last name</label>
          <select id="lname" v-model="lname" @change="setMaker">
            <option value="Alex">Alex</option>
            <option value="Michael">Michael</option>
            <option value="Ronald">Ronald</option>
          </select>
        </div>

        <div class="mx-1">
          <label for="prof">Select Profession</label>
          <select id="prof" v-model="prof" @change="setMaker">
            <option value="Programmer">Programmer</option>
            <option value="Carpenter">Carpenter</option>
            <option value="Driver">Driver</option>
          </select>
        </div>
      </div>
      <div class="mx-1">
        <div ref="editor" @dragover="editorOver" @drop="editorDrop"></div>
      </div>
    </div>

    <div>
      <span @dragstart="varDrag" draggable="true" class="vars" data-marker="fname" data-title="FIRST NAME" data-colour="warning">firstname</span>
      <span @dragstart="varDrag" draggable="true" class="vars" data-marker="lname" data-title="LAST NAME" data-colour="primary">last name</span>
      <span @dragstart="varDrag" draggable="true" class="vars" data-marker="prof" data-title="Profession" data-colour="secondary">profession</span>
    </div>
  </div>
  
  <div class="mx-1 mt-1">
      <button @click="previewer" id="preview">preview</button>
  </div>

  <p v-html="content" class="mx-1"></p>
</template>

<script>
import { QuillDeltaToHtmlConverter } from 'quill-delta-to-html'
import Quill from 'quill';
import '@vueup/vue-quill/dist/vue-quill.snow.css';

export default {
  data() {
    return {
      editor: null,
      fname: null,
      lname: null,
      prof: null,
      content: null,
      testing: 'this is a test {{ fname }}'
    };
  },

  watch: {
    fname: 'updateFnameVar',
    lname: 'updateLnameVar',
    prof: 'updateProfVar'
  },

  mounted() {
    var _this = this;
    var Embed = Quill.import('blots/embed');

    class TemplateMarker extends Embed {
        static create(value) {
            let node = super.create(value);
            
            node.setAttribute('class', 'badge badge-' + value.colour + ' var-' + value.cl);
            //Set up the badge, and badge colour
            
            node.setAttribute('data-marker', value.marker);
            //The marker is the $ rel_table[id] reference
            
            node.setAttribute('data-title', value.title);
            //
            
            node.innerHTML = value.title;
            //The title is what the user sees in their editor
            
            return node;
        }

        static value(node) {
            return {
                marker: node.getAttribute('data-marker'),
                title: node.getAttribute('data-title'),
            };
        }
    }

    TemplateMarker.blotName = 'TemplateMarker';
    TemplateMarker.tagName = 'span';

    Quill.register({
        'formats/TemplateMarker': TemplateMarker
    });

    this.quill = new Quill(this.$refs.editor, {
        modules: {
            toolbar: ['bold', 'italic', 'underline', 'strike']
        },
        placeholder: 'Please type in something...',
        theme: 'snow'
    });
  },

  methods: {
    editorOver(event) {
      event.preventDefault();
    },
    editorDrop(event) {
      let range = this.quill.getSelection(true);
      this.quill.insertText(range.index, ' ', Quill.sources.USER);
      this.quill.setSelection(range.index + 2);
      this.quill.insertEmbed(
        range.index,
        'TemplateMarker',
        {
          colour: event.dataTransfer.getData('colour'),
          marker: event.dataTransfer.getData('marker'),
          title: event.dataTransfer.getData('title'),
          cl: event.dataTransfer.getData('cl')
        },
      );
    },
    varDrag(event) {
      event.target.setAttribute('draggable', true)
      if (event.target.dataset.marker == 'fname') {
        event.dataTransfer.setData('marker', this.fname)
      } else if (event.target.dataset.marker == 'lname') {
        event.dataTransfer.setData('marker', this.lname)
      } else {
        event.dataTransfer.setData('marker', this.prof)
      }
      event.dataTransfer.setData('colour', event.target.dataset.colour)
      event.dataTransfer.setData('title',  event.target.dataset.title)
      event.dataTransfer.setData('cl',  event.target.dataset.marker)
    },
    previewer () {
      var delta = this.quill.getContents();
      var converter = new QuillDeltaToHtmlConverter(delta.ops, {});
      converter.renderCustomWith(function(customOp, contextOp){
          if (customOp.insert.type === 'TemplateMarker') {
              let val = customOp.insert.value;
              return val.marker;
          }
      });
    
      this.content = converter.convert();
    },
    updateFnameVar() {
      if (document.querySelectorAll('.var-fname')) {
        document.querySelectorAll('.var-fname').forEach(el=>{
          el.dataset.marker = this.fname
        })
      }
    },
    updateLnameVar() {
      if (document.querySelectorAll('.var-lname')) {
        document.querySelectorAll('.var-lname').forEach(el=>{
          el.dataset.marker = this.lname
        })
      }
    },
    updateProfVar() {
      if (document.querySelectorAll('.var-prof')) {
        document.querySelectorAll('.var-prof').forEach(el=>{
          el.dataset.marker = this.prof
        })
      }
    }
  },
}
</script>

<style>
  .ql-editor {
    height: 30vh
  }
</style>