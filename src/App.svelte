<main class="container">
    <hgroup>
        <h1>MakeCode Block Images</h1>
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
                <input type="text" name="projectUrl" bind:value={packageIdStrURL} />
                <small>MakeCode Projekt-URL über die Teilen funktion erstellen und hier eingeben, um dort geladene Erweiterungen verwenden zu können oder das ganze Projekt zu Rendern.</small>
            </label>
    
        </div>
  
        <label for="code">
            Code:
            <textarea name="code" bind:value={code}></textarea>
            <small>Leer lassen und oben einen MakeCode Projekt Link angeben, um ein ganzes projekt zu rendern.</small>
        </label>
  
    </form>


    <div class="grid">
        <a download="bild.svg" bind:this={svgLink}><button>SVG Herunterladen</button></a> 
        <a download="bild.png" bind:this={pngLink}><button>PNG Herunterladen</button></a>
    </div>


    <div class="item codeImage">
        <div class="grid">
            <div class="image-container"></div>
            <div class="image-containerPNG" bind:this={pngContainer}></div>
        </div>
	</div>

</main>

<pre class="makecodeSnippet lds-dual-ring" id="snippet_1">
    {code}
</pre>

<ThemeToggle />

<style>

	textarea {
		width: 100%;
		height: 10em;
        font-family: monospace;
	}

	.makecodeSnippet {
		display: none;
	}

</style>

<script>
    import { tick } from 'svelte';
    import ThemeToggle from './ThemeToggle.svelte';
        
    let code = ``; // code to be rendered as png
    $: codeToRender = (code.length < 2) ? "" : code; // if code is empty, set to empty string
    let packageIdStrURL = 'https://makecode.calliope.cc/_X3KdiDKFifpm'; // MakeCode Project url
    $: packageIdStr = packageIdStrURL.substring(packageIdStrURL.lastIndexOf("/") + 1); // get project id from url
    let snippetModeBool = true; // if true, remove surrounding start block
    let scalePng = 1; // faktor to scale png image
    let svgCode; 
    
    let pngContainer; // container for png image
    let pngLink; // download link to png image
    let svgLink; // download link to svg image

    $: renderSnippets(codeToRender, packageIdStr, snippetModeBool, scalePng)
    
    //// MakeCode Renderer
        var makeCodeRenderPre = makeCodeRenderPre || (function () {
        // pre waiting to be rendered
        // when undefined, iframe is loaded and ready
        var pendingPres = [];
        function injectRenderer() {
            var f = document.getElementById("makecoderenderer");
            // check iframe already added to the DOM
            if (f) {
                return;
            }
            var f = document.createElement("iframe");
            f.id = "makecoderenderer";
            f.style.position = "absolute";
            f.style.left = 0;
            f.style.bottom = 0;
            f.style.width = "1px";
            f.style.height = "1px";
            f.src = "https://makecode.calliope.cc/--docs?render=1"
            document.body.appendChild(f);
        }
    
        function renderPre(pre) {
            // console.log('render ' + pre.id)
            var f = document.getElementById("makecoderenderer");
            // check if iframe is added and ready (pendingPres is undefined)
            if (!f || !!pendingPres) {
                // queue up
                pendingPres.push(pre);
                injectRenderer();
            } else {
                f.contentWindow.postMessage({
                    type: "renderblocks",
                    id: pre.id,
                    code: codeToRender,
                    options: {
                        packageId: packageIdStr,
                        snippetMode: snippetModeBool
                    }
                }, "https://makecode.calliope.cc/");
            }
        }
    
        // listen for messages
        window.addEventListener("message", function (ev) {
            var msg = ev.data;
            if (msg.source != "makecode") return;
    
            // console.log(msg.type)
            switch (msg.type) {
                case "renderready":
                    // flush pending requests            				
                    var pres = pendingPres;
                    // set as undefined to notify that iframe is ready
                    pendingPres = undefined;
                    pres.forEach(function (pre) { renderPre(pre); })
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
                    code.parentElement.querySelector('.image-container').innerHTML = '';
                    pngContainer.innerHTML = '';
                    code.parentElement.querySelector('.image-container').appendChild(img);
                    
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
        // TODO ADD RENDER LOGIC HERE
        let pres = document.getElementsByClassName("makecodeSnippet");
        Array.prototype.forEach.call(pres, function (pre) {
            makeCodeRenderPre(pre);
        })
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