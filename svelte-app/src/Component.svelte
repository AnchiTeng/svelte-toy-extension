<script>
    import { parse } from "svelte/compiler";
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    // import * as d3 from "https://cdn.skypack.dev/d3@7"; can't use internal module


    let svelteFiles = []
    let astArray = []
   //let data=[5,10,15];
    let singleData;
    let  componentName;
    let tree = {};
    let treeObjs = {};
    
    
    onMount(async () => {
        await chrome.devtools.inspectedWindow.getResources(resources => {
            svelteFiles = resources.filter(resource => resource.url.includes('.svelte'))
            console.log('svelteFile is here line 16',svelteFiles);
            //get file url
            let fileName = svelteFiles[svelteFiles.length-1].url.slice(0,-7);
            console.log('filename--- ',fileName)//url
            let ans=[]

           for(let i=fileName.length-1;i>=0;i--){

  
              if(fileName[i] !=='/'){
              ans.push(fileName[i])
               }else{
                console.log('>>>>> line 32',ans)
                tree.top = ans.reverse().join('') ;
                break
               }

            }
           
            svelteFiles.forEach(file => {

            let parent = file.url.slice(0,-7);
            
            let ans=[]
            let parentName;

           for(let i=parent.length-1;i>=0;i--){

  
              if(parent[i] !=='/'){
              ans.push(parent[i])
               }else{
                console.log('>>>>>',ans)
                parentName = ans.reverse().join('') ;
                treeObjs[parentName]=[];
                break
               }

            }
            console.log('treeObjs',treeObjs)

                file.getContent(source => {
                    const ast = parse(source)
                    console.log('file line 20',file)
                    console.log("ast is here line 21",ast)
                    const parsedAST = parseAST(ast)
                    const astObj = {name: file.url, astData: parsedAST,parent:parentName}
                    astArray = [...astArray, astObj] 
                    
                    
                })
            })
        })
    })
    
    
    function parseAST(ast) {
        let children = []
        let state = []
        let store = []
        let props = []
        let reactives = []
        if(ast.instance){
        ast.instance.content.body.forEach(declaration => {
            switch(declaration.type) {
                case "ImportDeclaration": {
                    if (declaration.source.value.includes('.svelte')) {
                        console.log('source value >> ',declaration.source.value)
                        children = [...children, declaration.specifiers[0].local.name]
                       
                        
                        componentName = `${declaration.source.value.slice(2, declaration.source.value.length - 7)}`;
                        console.log('>>>',componentName)

                       

                        
                        
                    } else if (declaration.source.value.includes('stores.js')) {
                        store = [...store, declaration.specifiers[0].local.name]
                    }
                    break
                }
                case "VariableDeclaration": {
                    if (!declaration.declarations[0].init || (declaration.declarations[0].init.type === "Literal")) {
                        state = [... state, declaration.declarations[0].id.name]
                    }
                    break
                }
                case "ExportNamedDeclaration": {
                    if (declaration.declaration.type !== "FunctionDeclaration") {
                        props = [...props, declaration.declaration.declarations[0].id.name]
                    }
                    break
                }
                case "LabeledStatement": {
                     if (declaration.body.type === "ExpressionStatement" && declaration.body.expression.type === "AssignmentExpression") {
                         reactives = [...reactives, declaration.body.expression.left.name]
                     }
                     break
                }
            }
        })
    }
        // data = [5,10,15];
        //singleData=children[0];

        const astData = {
                children,
                state,
                store,
                props,
                reactives
        }

        console.log('astData',astData)

        return astData
    }
    
    

    function getTree(ast){

    console.log('astArr',astArray)
    console.log('astData',astArray.astData)
    astArray.forEach(item=>{
        console.log('name',item.name)

        if(treeObjs[item.name]){
                            treeObjs[item]=[...ast.astData.children]
                        }
                        
    })
    console.log('treeObjs',treeObjs)
    return treeObjs
    }
    

 

    


    let count = 1;
	// the `$:` means 're-run whenever these values change'
	$: doubled = count * 2;
	$: quadrupled = doubled * 2;

	function handleClick() {
		count += 1;
    }
     
    let childBtn;//got undefined 11/16 
   
    function getChildren(e){
        console.log('e >>',e.target.id)
        let id = e.target.id
      if(treeObjs[id]){
        childBtn=[...treeObjs[id]]
      }
      console.log("btn",childBtn)
      
      childBtn.forEach(item=>{
          console.log('item on 215',item)
        let newBtn = document.createElement('BUTTON');
          let t = document.createTextNode(`${item}`);
          newBtn.appendChild(t)
        let renderBtn= document.body.appendChild(newBtn);
        renderBtn.setAttribute('id',`${item}`)
        //renderBtn.setAttribute('onclick',`${getChildren}`);
        //# forEach will block inline event handlers, need to use addEventListenr
        renderBtn.addEventListener('click', getChildren);
        
          
      })
      
      
    }
    console.log('check btn',childBtn)

    //test collasible
let coll = document.getElementsByClassName("collapsible");
let i;

function collapsible(){
    let boo=0;
let soo=0
for (i = 0; i < coll.length; i++) {
  coll[i].addEventListener("click", function() {
    // this.classList.toggle("active");
    let content = this.nextElementSibling;
    if (content.style.display === "block") {
        boo+=1;
     content.style.display = "none";
    } else {
        soo++
      content.style.display = "block";
    }
  });
}
console.log('times-b-s',boo,soo);

}
 
 
</script>

<style>
   .collapsible {
  background-color: #777;
  color: white;
  cursor: pointer;
  padding: 18px;
  width: 100%;
  border: none;
  text-align: left;
  outline: none;
  font-size: 15px;
}

/* .active, .collapsible:hover {
  background-color: #555;
} */

.content {
  padding: 0 18px;
  display: none;
  overflow: hidden;
  background-color: #f1f1f1;
}
    .test{
        color:rgb(19, 0, 128);
    }
    </style>

<!-- <style>
	.chart div {
    font: 10px;
    background-color: steelblue;
    text-align: right;
    padding: 3px;
    margin: 1px;
    color:green;
  }
</style> -->
   
   
  <main>
    <!-- <p>see d3 effect next line</p>
    <div class="chart">
        {#each data as d}
          <div style="width: {d}px">
            {d}
          </div>
        {/each}
    </div>
    <div bind:this={el1} class="chart"></div>
    <div bind:this={el2} class="chart"></div> -->
    
    <!-- <div>
        <p>Here is testing children1 {data}</p>
        <button>{singleData}</button>
    </div> 

     <div>
        <h2> Top: {tree.top}</h2>
        
        </div> -->
        

    <button on:click={handleClick}>
        Count: {count}
    </button>
    <p>{count} * 2 = {doubled}</p>
    <p>{doubled} * 2 = {quadrupled}</p>
    <img alt='' src='logo.png'>
    <p>get file {componentName} </p>
    




<div>
    <ul>
        {#each astArray as ast}
            <li>{ast.name} && parent >> {ast.parent} </li>
            <ul>
                {#if ast.astData.children.length}
               
                <!-- <li>Children test  </li> -->
                <button type="button" class="collapsible" on:click={collapsible}>Children</button>
                <!-- <div class="content">{treeObjs[ast.parent]=[...ast.astData.children]}</div> -->
                 <!-- assign children to parent in treeObj and make this <p> dispaly hidden-->
                <p style="display:none;"> {treeObjs[ast.parent]=[...ast.astData.children]}</p> 
                 
                <ul class='content'>
                    {#each ast.astData.children as child}
                        <li>{child}</li>
                    {/each}
                </ul>
                {/if}
            </ul>
            <ul>
                {#if ast.astData.state.length}
                <!-- <li>State Variables 3</li> -->
                <button type="button" class="collapsible" on:click={collapsible}>State Variables</button>
                <ul class='content'>
                    {#each ast.astData.state as element}
                        <li>{element}</li>
                    {/each}
                </ul>
                {/if}
            </ul>
            <ul>
                {#if ast.astData.store.length}
                <!-- <li>Store Variables</li> -->
                <button type="button" class="collapsible" on:click={collapsible}>Store Variables</button>
                <ul class='content'>
                    {#each ast.astData.store as element}
                        <li>{element}</li>
                    {/each}
                </ul>
                {/if}
            </ul>
            <ul>
                {#if ast.astData.props.length}
                <!-- <li>Props</li> -->
                <button type="button" class="collapsible" on:click={collapsible}>Props</button>
                <ul class='content'>
                    {#each ast.astData.props as prop}
                        <li>{prop}</li>
                    {/each}
                </ul>
                {/if}
            </ul> <ul>
                {#if ast.astData.reactives.length}
                <!-- <li>Reactive Variables</li> -->
                <button type="button" class="collapsible" on:click={collapsible}>Reactive Variables</button>
                <ul class='content'>
                    {#each ast.astData.reactives as element}
                        <li>{element}</li>
                    {/each}
                </ul>
                {/if}
            </ul>
            <br>
            <hr>
        {/each}
    </ul>
    

</div>

<div>
    
    <button id={tree.top} on:click={getChildren}>{tree.top}</button>
    <p class= 'test' >using point to App directly</p>
    <button id='App' on:click={getChildren}>App</button>
   
    
    
  
   
</div>


</main>



