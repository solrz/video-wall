<Page name="home">
  <!-- Top Navbar -->
  <Navbar large sliding={false}>
    <NavLeft>
      <Link iconIos="f7:menu" iconAurora="f7:menu" iconMd="material:menu" panelOpen="left" />
    </NavLeft>
    <NavTitle sliding>video-wall</NavTitle>
    <NavRight>
      <Link iconIos="f7:menu" iconAurora="f7:menu" iconMd="material:menu" panelOpen="right" />
    </NavRight>
    <NavTitleLarge>video-wall</NavTitleLarge>
  </Navbar>
  <!-- Toolbar -->
  <Toolbar bottom>
    <Button onClick="{openUrlModal}">Change Src</Button>
    <Button onClick="{()=>window.dispatchEvent(new Event('resize'))}">Rescale</Button>
    <Button onClick="{changePlaySpeed}">Boost</Button>
    <Button external href="https://cors-anywhere.herokuapp.com/">Enable</Button>
  </Toolbar>
  <!-- Page content -->
  <Block inset>
    <div id="waterfall-grid" style="margin: 0px auto; ">
      {#each videos.slice(0,50) as v}
<!--        <p>{v.gsx$id.$t}</p>-->
<!--          <img style="width: 49%" src="https://drive.google.com/a/nctu.co/thumbnail?id={v.gsx$id.$t}" class="item"/>-->
        <video playbackRates="{[1,1.2,1.5,2]}" preload="none" controls playsinline  style="width: 49%" poster='https://drive.google.com/a/nctu.co/thumbnail?id={v.gsx$id.$t}' class="item">
          <source src="https://drive.google.com/uc?export=download&id={v.gsx$id.$t}" type="video/mp4">
<!--          <source src="https://www.learningcontainer.com/wp-content/uploads/2020/05/sample-mp4-file.mp4" type="video/mp4">-->
        </video>
<!--        <iframe src="https://drive.google.com/file/d/{v.gsx$id.$t}/preview"-->
<!--                frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope;picture-in-picture" allowfullscreen></iframe>-->
      {/each}
    </div>
  </Block>
</Page>
<script>
    import {
        Page,
        Navbar,
        NavLeft,
        NavTitle,
        NavTitleLarge,
        NavRight,
        Link,
        Toolbar,
        Block,
        BlockTitle,
        List,
        ListItem,
        Row,
        Col,
        Button, f7
    } from 'framework7-svelte';
  let imgs = '12ejio21jeoi21je1jejioio21jeio21jrdioj12r'
  let videos = []
    let currentPlaybackSpeed = 1
    let playbackRates = [1,1.2,1.5,2]
    function changePlaySpeed(){
        let i = playbackRates.indexOf(currentPlaybackSpeed) + 1
        currentPlaybackSpeed = playbackRates[i%playbackRates.length]
        document.querySelectorAll('video').forEach(v=>v.playbackRate = currentPlaybackSpeed)
    }
  function waterfall(container){
      if(typeof(container) === 'string')
          container = document.querySelector(container);

      // Freeze the list of nodes
      var els = [].map.call(container.children, function(el){
          el.style.position = 'absolute';
          return el;
      });
      container.style.position = 'relative';

      function margin(name, el){
          var style = window.getComputedStyle(el);
          return parseFloat(style['margin' + name]) || 0;
      }
      function px(n){ return n + 'px'; }
      function y(el){ return parseFloat(el.style.top) ; }
      function x(el){ return parseFloat(el.style.left); }
      function width(el){ return el.clientWidth; }
      function height(el){ return el.clientHeight; }
      function bottom(el){ return y(el) + height(el) + margin('Bottom', el); }
      function right(el){ return x(el) + width(el) + margin('Right', el); }

      function sort(l){
          l = l.sort(function(a, b){
              if(bottom(a) === bottom(b)){
                  return x(b) - x(a);
              }else{
                  return bottom(b) - bottom(a);
              }
          });
      }

      var boundary = [];

      // Deal with the first element.
      if(els.length){
          els[0].style.top = '0px';
          els[0].style.left = px(margin('Left', els[0]));
          boundary.push(els[0]);
      }

      // Deal with the first line.
      for(var i = 1; i < els.length; i++){
          var prev = els[i - 1],
              el = els[i],
              thereIsSpace = right(prev) + width(el) <= width(container);
          if(!thereIsSpace) break;
          el.style.top = prev.style.top;
          el.style.left = px(right(prev) + margin('Left', el));
          boundary.push(el);
      }

      // Place following elements at the bottom of the smallest column.
      for(; i < els.length; i++){
          sort(boundary);
          var el = els[i],
              minEl = boundary.pop();
          el.style.top = px(bottom(minEl) + margin('Top', el));
          el.style.left = px(x(minEl));
          boundary.push(el);
      }

      sort(boundary);
      var maxEl = boundary[0];
      container.style.height = px(bottom(maxEl) + margin('Bottom', maxEl));

      // Responds to window resize
      var containerWidth = width(container);
      function resize(e) {
          if(width(container) != containerWidth){
              e.target.removeEventListener(e.type, arguments.callee);
              waterfall(container);
          }
      }

      if (window.addEventListener) {
          window.addEventListener('resize', resize);
      } else {
          document.body.onresize = resize;
      }
  }

  import {onMount} from "svelte";
  onMount(function(){
      window.addEventListener("resize", ()=> {
          var grid = document.querySelector('#waterfall-grid');
          waterfall(grid)
      })
      loadVideos()
  })
  async function loadVideos(){
      let proxyUrl = 'https://cors-anywhere.herokuapp.com/';
      let sheetId = localStorage.getItem('sheetid')
      let url = proxyUrl + 'https://spreadsheets.google.com/feeds/list/' + sheetId + '/od6/public/values?alt=json';
      const resp = await fetch(url)
      const json = await resp.json()
      videos = json.feed.entry
      return shuffle(videos)
  }
  function shuffle(array) {
      var currentIndex = array.length, temporaryValue, randomIndex;

      // While there remain elements to shuffle...
      while (0 !== currentIndex) {

          // Pick a remaining element...
          randomIndex = Math.floor(Math.random() * currentIndex);
          currentIndex -= 1;

          // And swap it with the current element.
          temporaryValue = array[currentIndex];
          array[currentIndex] = array[randomIndex];
          array[randomIndex] = temporaryValue;
      }

      return array;
  }
  function openUrlModal(){
      f7.dialog.prompt('Change url',(v)=>{
          let endIndex = v.replace('https://docs.google.com/spreadsheets/d/','').indexOf('/');
          let sheetId = v.replace('https://docs.google.com/spreadsheets/d/','').slice(0,endIndex);
          localStorage.setItem('sheetid',sheetId)
          loadVideos()
      })
  }

</script>
