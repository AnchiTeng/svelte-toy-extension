<script>
    import { parse } from "svelte/compiler";
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    // import * as d3 from "https://cdn.skypack.dev/d3@7"; can't use internal module



    let svelteFiles = []
    let astArray = []
    let  componentName;
    let tree = {};
    let treeObjs = {};
    
    
    onMount(async () => {
        await chrome.devtools.inspectedWindow.getResources(resources => {
            svelteFiles = resources.filter(resource => resource.url.includes('.svelte'))
            console.log('svelteFile is here line 16',svelteFiles);
           
            //get file name
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
                    // console.log('file line',file)
                    // console.log("ast is here",ast)
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
        

        const astData = {
                children,
                state,
                store,
                props,
                reactives
        }

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

  // let color = ['green','red','yellow']

    //  let el1;
    // onMount(() => {
	// 	d3.select(el1)
	// 		.selectAll("button")
	// 		.data(color)
	// 		.enter()
	// 		.append("div")
	// 		.style("color", function(d) {
	// 			return 'red';
	// 		})
	// 		.text(function(d) {
	// 			return d;
	// 		});
	// });


let childBtn;//got undefined 11/16 
    

    function getChildren(e){
        let color = ['green','red','yellow','blue','purple','gray','orange','brown','green','red','yellow','blue','purple','gray','orange','brown'];
        console.log('e >>',e.target.id)
        let id = e.target.id
      if(treeObjs[id]){
        childBtn=[...treeObjs[id]]
      }
      console.log("btn",childBtn)
      let c=document.getElementById('treeBtns')
      childBtn.forEach(item=>{
          console.log('item on 215',item)
        let newBtn = document.createElement('BUTTON');
          let t = document.createTextNode(`${item}`);
          newBtn.appendChild(t);
          
        let renderBtn= c.appendChild(newBtn);
        renderBtn.setAttribute('id',`${item}`);

        d3.select(c)
          .selectAll('button')
          .data(color)
          .style("color", function(d) {
				return d;
			})

        //renderBtn.setAttribute('onclick',`${getChildren}`);
        //# forEach will block inline event handlers, need to use addEventListenr
        renderBtn.addEventListener('click', getChildren);
        
        
          
      })
      //insert <br/> after children rendered
    let brLine = document.createElement('BR');
        c.appendChild(brLine)
      
    }
    // console.log('check btn',childBtn)


let collapse = document.getElementsByClassName("collapsible");
let i;

function collapsible(){
 
  for (i = 0; i < collapse.length; i++) {
    collapse[i].addEventListener("click", function() {
    // this.classList.toggle("active");
      let content = this.nextElementSibling;
      if (content.style.display === "block") {
      
       content.style.display = "none";
      } else {
      
      content.style.display = "block";
      }
    });
  }
}

    
</script>


   <h1>In ComponentTree</h1>
   
   <div>
    <ul>
        {#each astArray as ast}
            <li class='parent'>Parent Component: {ast.parent} </li>
            <ul>
                {#if ast.astData.children.length}
               
               
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

<div id='treeBtns'>
   
    <p>Component Tree</p>
    <!-- Point to App  directly is not the accurate way since not every app using variable 'App' as the top Component; -->
    <button id='App' on:click={getChildren}>App</button>
    <br/>
   
    
    
  
</div>



<style>
.parent{
   background-color: #777;
   color: white;
   cursor: pointer;
   padding: 18px;
   width: 40%;
   border: none;
   text-align: center;
   outline: none;
   font-size: 15px;

}

.collapsible {
   background-color: rgb(64, 140, 240);
   color: white;
   cursor: pointer;
   padding: 12px;
   width: 40%;
   border: none;
   text-align: left;
   outline: 4px;
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
     
</style>