<template>
  <div id="highlightedDoc" v-html="sourceHTML">

  </div>
</template>

<script>
export default {
  name: 'RenderHTML',
  props: ['sourceHTML', 'offsetArr'],
  data(){
    return {
      counter: 0,
    }
  },
  methods: {
    getAbsoluteXPath(element) {
    var comp, comps = [];
    var parent = null;
    var xpath = '';
    var getPos = function(element) {
        var position = 1,
            curNode;
        if (element.nodeType == Node.ATTRIBUTE_NODE) {
            return null;
        }
        for (curNode = element.previousSibling; curNode; curNode = curNode.previousSibling) {
            if (curNode.nodeName == element.nodeName) {
                ++position;
            }
        }
        return position;
    };

    if (element instanceof Document) {
        return '/';
    }

    for (; element && !(element instanceof Document); element = element.nodeType == Node.ATTRIBUTE_NODE ? element.ownerElement : element.parentNode) {
        comp = comps[comps.length] = {};
        switch (element.nodeType) {
            case Node.TEXT_NODE:
                comp.name = 'text()';
                break;
            case Node.ATTRIBUTE_NODE:
                comp.name = '@' + element.nodeName;
                break;
            case Node.PROCESSING_INSTRUCTION_NODE:
                comp.name = 'processing-instruction()';
                break;
            case Node.COMMENT_NODE:
                comp.name = 'comment()';
                break;
            case Node.ELEMENT_NODE:
                comp.name = element.nodeName;
                break;
        }
        comp.position = getPos(element);
    }

    for (var i = comps.length - 1; i >= 0; i--) {
        comp = comps[i];
        xpath += '/' + comp.name.toLowerCase();
        if (comp.position !== null && comp.position !== 1) {
            xpath += '[' + comp.position + ']';
        }
    }
    return xpath;
    },
    highlight(){
      this.offsetArr.forEach(({xpath, offsets})=>{
        offsets.sort((a,b)=>(b-a)).reverse();
        let minOffset = Number.MAX_SAFE_INTEGER;
        const element = document.evaluate(xpath, document, null, XPathResult.FIRST_ORDERED_NODE_TYPE, null).singleNodeValue;
        let elHTML = element.innerHTML;
        let localStart;
        let localEnd;
        const colorPickerArray = [1,2,3];
        let currentColorIndex = 0;

        offsets.forEach(([start, end], index)=>{
          if(minOffset <= end){
            //Apply Overlapping logic
            let offsetShift = 0;
            for(let i=index-1;i>=0;i--){
              [localStart, localEnd] = offsets[i];
              if(start == localStart || (end > localStart && end < localEnd)) offsetShift+=22;
              if (end > localEnd) offsetShift+=29;
            }      
            end+=offsetShift;     
            elHTML = elHTML.slice(0,start) + `<span class="color-${colorPickerArray[currentColorIndex]}">` + elHTML.slice(start,end+1) + `</span>` + elHTML.slice(end+1);
            currentColorIndex = currentColorIndex == 2? 0:++currentColorIndex;
          } else {
            //Apply Regular Logic
            elHTML = elHTML.slice(0,start) + `<span class="color-0">` + elHTML.slice(start,end+1) + `</span>` + elHTML.slice(end+1);
          }
          minOffset = minOffset > start ? start:minOffset;
        });
        element.innerHTML = elHTML;
      });
    },
    sourceTagging(){
      document.getElementById("highlightedDoc").addEventListener("mouseup", ($event)=>{
        const selection = document.getSelection().getRangeAt(0);
        console.log(selection);
        this.offsetCompute(selection.commonAncestorContainer.previousSibling);
        let start = selection.startOffset + this.counter;
        let end = selection.endOffset + this.counter;
        this.counter = 0;
        console.log({
          start,
          end,
          xpath: this.getAbsoluteXPath($event.target)
        })
      })
    },
    offsetCompute(el){
      if(el == null) return;
      if(el.nodeName === "SPAN") {
        this.counter+=el.innerText.length;
      }
      else if(el.nodeName === "#text") {
        this.counter+=el.length;
      } else {
        this.counter+=el.outerHTML.length;
      }
      this.offsetCompute(el.previousSibling);
    }
  },
  mounted(){
    this.highlight();
    this.sourceTagging();
  }
}
</script>

>
<style>
#highlightedDoc {
  height: 80vh;
  width: 700px;
  margin-left: 490px;
  font-size: 2rem;
}
.color-0{
  background-color: rgb(238, 238, 190);
  border: 1px solid yellow;
}
.color-1{
  background-color: rgb(233, 165, 165);
  border: 1.9px solid red;
}
.color-2{
  background-color: rgb(175, 216, 216);
  border: 2.5px solid aqua;
}
</style>
