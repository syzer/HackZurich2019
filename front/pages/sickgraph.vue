<template>
  <v-layout>
    <v-flex class="text-center">
      <v-row align="center" justify="center">
        <span id="graph"></span>
      </v-row>
    </v-flex>
  </v-layout>
</template>
<script>
  import * as d3 from 'd3'

  export default {
    components: {},
    head() {
      return {
        script: []
      }
    },
    data() {
      const style = {
        red: '#EE0000',
        orange: '#FF6600',
        yellow: '#F6AD2B',
        green: '#60AD08',
        gray: '#CFCAC7'
      }

      return {
        style
      }
    },
    mounted() {
      // const socket = io()
      const imgDir = 'img/'
      const w = d3.select(window)[0][0]
      const width = w.innerWidth
      const height = w.innerHeight
      const nodeY = 30
      const layout = {
        linkDistance: 90,
        charge: -100
      }

      const svg = d3.select('#graph').append('svg')
        .attr('width', width)
        .attr('height', height)

      // http://stackoverflow.com/questions/16265123/resize-svg-when-window-is-resized-in-d3-js
      d3.select(window).on('resize.updatesvg', () => {
        const x = w.innerWidth || e.clientWidth || g.clientWidth
        const y = w.innerHeight || e.clientHeight || g.clientHeight

        svg.attr('width', x).attr('height', y)
      })

      const force = d3.layout.force()
        .gravity(0.02)
        .linkDistance(layout.linkDistance)
        .charge(layout.charge)
        .size([width / 2 - 100, height - 200])

      const getStatusColor = (statuses, name) => {
        const between = (x, min, max) => x >= min && x <= max
        const status = statuses.find(e => e.name === name)

        if (!status) {
          return this.style.gray
        }
        if (between(status.score, 0.9, 1)) {
          return this.style.green
        }
        if (between(status.score, 0.5, 0.9)) {
          return this.style.yellow
        }
        if (between(status.score, 0.3, 0.5)) {
          return this.style.orange
        }
        return this.style.red
      }

      const updateGraph = (statuses) => [
        svg.selectAll('.status-badge'),
        svg.selectAll('.status-pill')
      ].map(blaa =>
        blaa.transition()
          .duration(1500)
          .style('fill', (d, i) => getStatusColor(statuses, d.name)))

      const getStatuses = () => new Promise((res, rej) => {
        // TODO pull data
        // socket.on('state', statuses => {
        //   console.log('new state', statuses)
        //   res(statuses)
        //   updateGraph(statuses)
        // })
        res([
          { score: 1, name: 'Lukas Gintowt' },
          { score: 0.5, name: 'Attila Zimmermann' },
          { score: 0.1, name: 'Alexander Wood' },
          { score: 0.5, name: 'Eliza Zimmermann' },
          { score: 0.07407407407407407, name: 'tris' },
          { score: 0.2, name: 'ivan' },
          { score: 1, name: 'tony' },

          { score: 0, name: 'It department' },
          { score: 0.35714285714285715, name: 'eliza' },
          { score: 0.3333333333333333, name: 'maria' }
          ])
      })

      const getGraphData = () => new Promise((res, rej) =>
        d3.json('/graph.json', (err, data) =>
          err ? rej(err) : res(data)))

      const dblclick = d =>
        d3.select(d3.node).classed('fixed', d.fixed = false)

      const dragstart = d =>
        d3.select(d3.node).classed('fixed', d.fixed = true)

      Promise.all([
        getGraphData(),
        getStatuses()
      ]).then(data => {
        const [graphData, statuses] = data
        console.log(graphData, statuses)

        const drag = force.drag().on('dragstart', dragstart)

        const link = svg.selectAll('.link')
          .data(graphData.links)
          .enter()
          .append('line')
          .attr('class', 'link')
          .style('stroke-width', d => Math.sqrt(d.value))

        const tick = () => {
          link.attr('x1', d => d.source.x)
            .attr('y1', d => d.source.y)
            .attr('x2', d => d.target.x)
            .attr('y2', d => d.target.y)

          node.attr('cx', d => d.x)
            .attr('cy', d => d.y)
        }

        force.nodes(graphData.nodes)
          .links(graphData.links)
          .start()
          .on('tick', tick)

        function animLink(d) {
          d3.select(this)
            .transition()
            .delay(0)
            .duration(50000)
            .ease('linear')
            .attrTween('stroke-dashoffset', () =>
              (t) => d3.interpolateString('0', '1000')(t)
            )
            .each('end', animLink)
        }

        link.style('stroke-dasharray', '10,10')
          .each(animLink)

        let node = svg.selectAll('.node')
          .data(graphData.nodes)
          .enter()
          .append('g')
          .attr('class', 'node')
          .on('dblclick', dblclick)
          .call(force.drag)

        // status badge
        // See updateGraph()
        node.append('rect')
          .attr('class', 'status-badge')
          .attr('x', -28)
          .attr('y', -nodeY)
          .attr('width', 15)
          .attr('height', nodeY * 2.35)
          .attr('opacity', 1)
          .style('fill', (d, i) =>
            getStatusColor(statuses, d.name))

        node.append('rect')
          .attr('x', -20)
          .attr('y', -nodeY / 2 - 15)
          .attr('rx', 6)
          .attr('width', 270)
          .attr('height', 70)
          .attr('opacity', 1)
          .on('mouseover', function () {
            d3.select(this).classed('active', true)
          })
          .on('mouseout', function () {
            d3.select(this).classed('active', false)
          })

        node.append('svg:image')
          .attr('xlink:href', d => d.icon ? `${imgDir}${d.icon}` : 'favicon.ico')
          .attr('x', -15)
          .attr('y', -20)
          .attr('width', 50)
          .attr('height', 50)

        node.append('text')
          .attr('dx', 40)
          .attr('dy', nodeY / 2)
          .attr('class', 'name')
          .text(d => d.name)

        force.on('tick', () => {
          link.attr('x1', d => d.source.x)
            .attr('y1', d => d.source.y)
            .attr(  'x2', d => d.target.x)
            .attr('y2', d => d.target.y)

          node.attr('transform', d => `translate( ${d.x} , ${d.y} )`)
        })
      })
        .catch(console.error)
    }
  }
</script>
<!--<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css"-->
<!--integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">-->
<!--<script src="js/socket.io.min.js"></script>-->
<style>
  :root {
    --active-color: #c2751a;

    --status-green: #60AD08;
    --status-green-hover: #315903;
    --status-red: #EE0000;
    --status-red-hover: #BA0000;
    --status-yellow: #F6AD2B;
    --status-yellow-hover: #D98D06;
    --status-orange: #FF6600;
    --status-gray: #CFCAC7;

    --migros-orange: #ff6600;
    --migros-orange-hover: #DE450A;
    --text-black: #333333;
    --border-gray: #CFCAC7;
    --label-small-gray: #BBBBBB;
    --label-big-gray: #676767;
    --background-darker: #E7E5E3;
    --background-lighter: #F1EFEE;
  }

  body {
    /*padding: 50px;*/
    font: 12px "Lucida Grande", Helvetica, Arial, sans-serif;
    background-color: var(--background-lighter);


  }

  .theme--dark.v-application {
    /*background: #303030;*/
    background: #F1EFEE;
    color: #FFFFFF;
  }

  a {
    color: #00B7FF;
  }

  .link {
    stroke: var(--label-small-gray);
    stroke-width: 2;
  }

  .node {
    cursor: move;
  }

  .node text {
    font-family: HelveticaNeue, sans-serif;
    font-size: 22px;
    font-weight: bold;
  }

  .node > rect {
    fill: var(--label-small-gray);
  }

  .node > .active {
    fill: var(--active-color);
  }

  .node .name {
    pointer-events: none;
  }

  .description:hover {
    fill: var(--active-color);
  }

  .description {
    fill: white;
    background-color: var(--migros-orange);
  }

  .Rectangle {
    width: 327px;
    height: 70px;
    border-radius: 5px;
    background-color: var(--background-lighter);
    border: solid 1px var(--border-gray);
  }

</style>
