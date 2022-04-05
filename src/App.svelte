<main class="container">
    <hgroup>
        <h1>MakeCode Block Bilder für
            <select id="fruit" required style='display:inline-block; width:auto' bind:value={editorURL}>
              <option value='https://makecode.calliope.cc/' selected>Calliope</option>
              <option value='https://makecode.microbit.org/'>Microbit</option>
            </select>
        </h1>
        <h2>Mit diesem Tool können die Blöcke aus MakeCode anhand von Code als Bilder in SVG und als PNG erstellt werden.</h2>
    </hgroup>

    <form>

        <div class="grid">
            <div class="grid">
    
                <label for="snippet">
                    Snippet Modus<br>
                    <input type="checkbox" name="snippet" bind:checked="{snippetModeBool}" role="switch" style="display:block; margin:32px 0px;" />
                    <small>Umschließenden Start-Block entfernen</small>
                </label>
            
                <label for="scalePng">
                    PNG Skalieren
                    <input type="number" name="scalePng" min=0 bind:value={scalePng} />
                    <small>Mit einem größeren Wert erhält man ein großeres PNG Bild.</small>
                </label>

            </div>

            <label for="projectUrl">
                MakeCode Projekt Link (Optional)
                <input type="text" name="projectUrl" placeholder="https://makecode.calliope.cc/..." bind:value={packageIdStrURL} />
                <small>MakeCode Projekt-URL über die Teilen Funktion erstellen und hier eingeben, um dort geladene Erweiterungen verwenden zu können oder das ganze Projekt zu rendern.</small>
            </label>
    
        </div>
  
        <label for="code">
            Code
            <textarea name="code" bind:value={code} placeholder="basic.showIcon(IconNames.Heart) ..."></textarea>
            <small>Leer lassen und oben einen MakeCode Projekt Link angeben, um ein ganzes Projekt zu rendern.</small>
        </label>
  
    </form>


    <div class="grid">
        <a download="bild.svg" bind:this={svgLink}><button>SVG herunterladen</button></a> 
        <a download="bild.png" bind:this={pngLink}><button>PNG herunterladen</button></a>
    </div>

    <div class="item codeImage">
        <div class="grid">
            <div class="image-container" bind:this={svgContainer}></div>
            <div class="image-containerPNG" bind:this={pngContainer}></div>
        </div>
	</div>

    <hr style="margin: 50px 0px;">
    <a href="https://github.com/calliope-edu/MakeCode-Block-Image-Tool" target="_blank">
        <img src="https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white" alt="GitHub-Logo" />
    </a>

    <a href="https://github.com/calliope-edu/MakeCode-Block-Image-Tool" target="_blank">
        <img src="https://badgen.net/github/last-commit/calliope-edu/MakeCode-Block-Image-Tool" alt="GitHub last commit" />
    </a>  

    <a href="https://github.com/calliope-edu/MakeCode-Block-Image-Tool/blob/master/LICENSE" target="_blank">
        <img src="https://img.shields.io/github/license/calliope-edu/MakeCode-Block-Image-Tool.svg" alt="MIT License" />
    </a>

    <a href="https://github.com/calliope-edu/MakeCode-Block-Image-Tool/issues" target="_blank">
        <img src="https://img.shields.io/github/issues/calliope-edu/MakeCode-Block-Image-Tool.svg" alt="GitHub open issues" />
    </a>

</main>

<pre class="makecodeSnippet lds-dual-ring" id="snippet_1" bind:this={codePre}>
    {code}
</pre>

<iframe bind:this={iframeHandler} src='{editorURL}--docs?render=1' style="position: absolute; left: 0; bottom: 0; width: 1px; height: 1px; border: none;"></iframe>

<ThemeToggle />

<style>

	textarea {
		width: 100%;
		height: 12em;
        font-family: monospace;
	}

	.makecodeSnippet {
		display: none;
	}

    ::placeholder {
        opacity: .3;
    }

</style>

<script>
    import { tick } from 'svelte';
    import ThemeToggle from './ThemeToggle.svelte';
        
    let code = ``; // code to be rendered as png
    $: codeToRender = (code.length < 2) ? "" : code; // if code is empty, set to empty string
    let packageIdStrURL = ''; // MakeCode Project url
    $: packageIdStr = packageIdStrURL.substring(packageIdStrURL.lastIndexOf("/") + 1); // get project id from url
    let snippetModeBool = true; // if true, remove surrounding start block
    let scalePng = 1; // faktor to scale png image
    let svgCode; 
    
    let codePre;
    let iframeHandler;
    let svgContainer;
    let pngContainer; // container for png image
    let pngLink; // download link to png image
    let svgLink; // download link to svg image

    let editorURL = "https://makecode.calliope.cc/";

    $: renderSnippets(codeToRender, packageIdStr, snippetModeBool, scalePng, editorURL, code)
    
    //// MakeCode Renderer
        var makeCodeRenderPre = makeCodeRenderPre || (function () {
    
        function renderPre() {
            iframeHandler.contentWindow.postMessage({
                type: "renderblocks",
                id: codePre.id,
                code: codeToRender,
                options: {
                    packageId: packageIdStr,
                    snippetMode: snippetModeBool
                }
            }, editorURL);
        }
    
        // listen for messages
        window.addEventListener("message", function (ev) {
            var msg = ev.data;
            if (msg.source != "makecode") return;
    
            // console.log(msg.type)
            switch (msg.type) {
                case "renderready":
                    renderPre();
                    break;
                case "renderblocks":
                    var svg = msg.svg; // this is an string containing SVG
                    var id = msg.id; // this is the id you sent
                    svgCode = svg.replace(/,\.blocklyDropDownContent \.blocklyPianoDiv.*\]\]><\/style>/gims, "]]></style>");
                    var svgUri = "data:image/svg+xml;charset=utf-8," + encodeURIComponent(svgCode);
                    // replace text with svg
                    var img = document.createElement("img");
                    svgLink.href=svgUri;
                    img.src = svgUri;
                    img.width = msg.width;
                    img.height = msg.height;
                    var code = document.getElementById(id);
                    svgContainer.innerHTML = '';
                    pngContainer.innerHTML = '';
                    svgContainer.appendChild(img);
                    
                    svgToPng(svg,(imgData)=>{
                        const pngImage = document.createElement('img');
                        pngContainer.innerHTML = '';
                        pngContainer.appendChild(pngImage);
                        pngImage.src=imgData;
                        pngLink.href = imgData;
                        
                    });
                    break;
            }
        }, false);
    
        return renderPre;
        })();
    
        async function renderSnippets() {
            await tick();
            makeCodeRenderPre(codePre);
        }
    //// END MakeCode Renderer
    
    
    /// SVG to PNG
        function svgToPng(svg, callback) {
            const url = getSvgUrl(svg);
            svgUrlToPng(url, (imgData) => {
                callback(imgData);
                URL.revokeObjectURL(url);
            });
        }
        function getSvgUrl(svg) {
            return  URL.createObjectURL(new Blob([svg], { type: 'image/svg+xml' }));
        }
        function svgUrlToPng(svgUrl, callback) {
            const svgImage = document.createElement('img');
            // imgPreview.style.position = 'absolute';
            // imgPreview.style.top = '-9999px';
            // pngContainer.innerHTML = '';
            pngContainer.appendChild(svgImage);
            svgImage.onload = function () {
                const canvas = document.createElement('canvas');
                canvas.width = svgImage.clientWidth * scalePng;
                canvas.height = svgImage.clientHeight * scalePng;
                const canvasCtx = canvas.getContext('2d');
                canvasCtx.drawImage(svgImage, 0, 0, canvas.width, canvas.height);
                const imgData = canvas.toDataURL('image/png');
                callback(imgData);
                // pngContainer.removeChild(imgPreview);
            };
            svgImage.src = svgUrl;
        }
    /// END SVG to PNG

</script>