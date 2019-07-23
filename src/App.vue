<template>
  <main>
    <div class="top-nav">
      <div class="nav-items">
<!--        <span class="nav-item hoverable mobile-only">Sequencer</span>
        <span class="nav-item hoverable mobile-only">Instruments</span> -->
        <span class="nav-item hoverable">Status: </span>
        <span class="nav-item hoverable">{{status}}</span>
        <br class="mobile-only">
        <span class="logo">
          <h2 class="header">
            <span 
              v-for="(c, i) in title"
              @click="wierd(i)"
              class="hoverable">{{c}}</span>
          </h2>
        </span>
      </div>
    </div>
    <div class="side-panel instruments">
      <div class="content">
        <div 
          :class="`sub-header hoverable ${infoState == 'sequencer' ? 'active' : ''}`" 
          @click="infoState = 'sequencer'"
        >
          Sequencer
        </div>
        <ul v-if="infoState == 'sequencer'">
          <li 
            v-for="(value, key) in settings"
          >
            <span class="setting">
              <span class="hoverable">{{key}}:</span>
              <input class="setting-input" v-model="settings[key]">
            </span>
          </li>
          <li>
            <span 
              class="hoverable" 
              @click="exandExportOpts = !exandExportOpts">export</span>
            <ul v-if="exandExportOpts">
              <li>
                <span class="setting">
                  <span class="hoverable">loops count:</span>
                  <input class="setting-input" v-model="loopsCount">
                </span>
              </li>
              <li>
                <span class="hoverable" @click="writeWav">wav</span>
              </li>
            </ul>

          </li>
          <li>
            <span 
              v-if="!isPlaying"
              class="hoverable" 
              @click="play">play!</span>
            <span 
              v-if="isPlaying"
              class="hoverable" 
              @click="stop">stop!</span>
          </li>
        </ul>

        <div 
          :class="`sub-header hoverable ${infoState == 'instruments' ? 'active' : ''}`" 
          @click="infoState = 'instruments'"
        >
          Instruments
        </div>

        <ul v-if="infoState == 'instruments'">
          <li 
            v-for="(instrument, i) in instruments"
          >
            <span 
              class="instr-name hoverable" 
              @click="expandInstrOpts = expandInstrOpts == i ? -1 : i"
            >
              {{toHex(i)}}: {{instrument.name}}
            </span>
            <ul v-if="expandInstrOpts == i" class="instr-opts">
              <li>
                <span 
                  class="hoverable" 
                  @click="editInstr = editInstr == i ? -1 : i"
                >
                  edit
                </span>
                <ul v-if="editInstr == i">
                  <li 
                    v-for="(value, key) in instrument"
                  >
                    <span class="setting">
                      <span class="hoverable">{{key}}:</span>
                      <input class="setting-input" v-model="instrument[key]">
                    </span>
                  </li>
                </ul>
              </li>
              <li>
                <span class="hoverable" @click="deleteInstr(i)">delete</span>
              </li>
              <li>
                <span class="hoverable" @click="dublicate(i)">dublicate</span>
              </li>
              <li>
                <span class="hoverable" @click="test(instrument)">play!</span>
              </li>
            </ul>
          </li>
          <li>
            <span class="instr-name hoverable" @click="addInstrument">add new</span>
          </li>
        </ul>
      </div>
    </div>
    <div class="side-panel info-area">
      <div class="content">
        <div class="container">
          <pre ref="background" class="editor-background" v-html="sequence"></pre>
          <pre ref="highlights" class="highlights" v-html="highlights"></pre>
          <textarea rows="16" class="editor" v-model="sequence" @input="handleInput" @scroll="handleScroll" ref="textarea">
          </textarea>
        </div>
      </div>
    </div>
  </main>
</template>

<style type="text/css">
  .header {
    display: inline;
  }

</style>

<script>
  import './assets/sequencer.css'

  import md5 from 'md5'
  import wavencoder from 'wav-encoder'
  import { saveAs } from 'file-saver';

  const synthkit = require('synthkit')


  export default {
    head: {
      title: 'ascii-sequencer',
      meta: [
        {
          'vmid': 'og:title',
          'property': 'og:title',
          'content': 'ascii-sequencer',
          'template': chunk => `${chunk} :: lurking-exorcist`
        },
        { hid: 'description', name: 'description', content: 'very minimalistic sequencer, write music with ascii-characters' }
      ]
    },
    data: () => ({
      title: 'ascii-sequencer (beta)',
      normalTitle: 'ascii-sequencer (beta)',
      wierdTitle: '4sc11-s3рu3nc3г [ь3t4]',
      settings: {
        tempo: 120,
        beat: 8
      },
      instruments: [
        {
          "name":"kick",
          "osc":"sin",
          "gain":"3",
          "note":"A2",
          "freq":55,
          "attack":"0.1",
          "decay":"15",
          "sustain":"0",
          "release":"10",
          "filterType":"lpf",
          "cutoff":2000,
          "resonance":"1",
          "time":"500"
        },
        {
          "name":"hihat",
          "osc":"noise",
          "gain":"0.1",
          "note":"C6",
          "freq":523.2511306011972,
          "attack":"0",
          "decay":"6",
          "sustain":"0",
          "release":"100",
          "filterType":"bpf",
          "cutoff":"7000",
          "resonance":"1",
          "time":"500"
        },
        {
          "name":"snare",
          "osc":"noise",
          "gain":"4",
          "note":"C6",
          "freq":523.2511306011972,
          "attack":"0",
          "decay":"10",
          "sustain":"0",
          "release":"100",
          "filterType":"lpf",
          "cutoff":"2500",
          "resonance":"0.01",
          "time":"500"
        },
        {
          "name":"note_1",
          "osc":"triangle",
          "gain":"1",
          "note":"A5",
          "attack":"5",
          "decay":"400",
          "sustain":"0",
          "release":"200",
          "filterType":"lpf",
          "cutoff":2000,
          "resonance":"1",
          "time":"500"
        },
        {
          "name":"note_2",
          "osc":"triangle",
          "gain":"1",
          "note":"C6",
          "attack":"5",
          "decay":"400",
          "sustain":"0",
          "release":"200",
          "filterType":"lpf",
          "cutoff":2000,
          "resonance":"1",
          "time":"500"
        },
        {
          "name":"note_3",
          "osc":"triangle",
          "gain":"1",
          "note":"D6",
          "attack":"5",
          "decay":"400",
          "sustain":"0",
          "release":"200",
          "filterType":"lpf",
          "cutoff":2000,
          "resonance":"1",
          "time":"500"
        },
        {
          "name":"note_4",
          "osc":"triangle",
          "gain":"1",
          "note":"E6",
          "attack":"5",
          "decay":"400",
          "sustain":"0",
          "release":"200",
          "filterType":"lpf",
          "cutoff":2000,
          "resonance":"1",
          "time":"500"
        },
        {
          "name":"note_5",
          "osc":"triangle",
          "gain":"1",
          "note":"G6",
          "attack":"5",
          "decay":"400",
          "sustain":"0",
          "release":"200",
          "filterType":"lpf",
          "cutoff":2000,
          "resonance":"1",
          "time":"500"
        }
      ],

      infoState: 'instruments',
      expandInstrOpts: -1,
      editInstr: -1,
      sequence: '02..02..02..02..02..02..02..02..02..02..02..02..02..02..02..02..\n01..............01....01........01..............01....01........\n........03..............03..............03..............03......\n04......05..06..........04....0408......07....0605..............',
      highlights: '',

      audioCtx: null,
      gainNode: null,
      synthNode: null,

      isPlaying: false,
      status: 'ready.',

      loopsCount: 4,

      exandExportOpts: false
    }),

    watch: {
      instruments: {
          handler(val){
            localStorage.setItem('instruments', JSON.stringify(this.instruments))
          },
          deep: true
      },

      sequence(val){
        localStorage.setItem('sequence', this.sequence)
      }
    },

    mounted(){
      try{
        const sequence = localStorage.getItem('sequence');
        const instrs = JSON.parse(localStorage.getItem('instruments'));

        if(instrs)
          this.instruments = instrs;

        if(sequence)
          this.sequence = sequence;

        this.handleInput();
      }catch(e){
        console.log(e);
      }
    },

    methods: {
      wierd(i){
        const tittl = this.title.split('');

        if(tittl[i] == this.wierdTitle[i]){
          tittl[i] = this.normalTitle[i];
        }else{
          tittl[i] = this.wierdTitle[i];
        }
        
        this.title = tittl.join('');
      },
      checkUniq(name){
        const index = this.instruments.findIndex(inst => inst.name == name);

        return index;
      },
      dublicate(i){
        let _name = this.instruments[i].name;
        let name = _name;

        let index  = this.checkUniq(_name);

        while(index != -1){
          name = name.split('');

          const lastN = this.instruments[index].name.slice(name.length-1);

          if(!isNaN(+lastN)){
            name[name.length-1] = +lastN + 1;
          }else{
            name.push('_1');
          }
          name = name.join('')
          index = this.checkUniq(name);
        }

        this.instruments.push(
          Object.assign({}, this.instruments[i], { name })
        );
      },
      deleteInstr(i){
        this.instruments.splice(i, 1);
      },
      applyHighlights(text) {
        text = text
          .replace(/\n/g, '<br>')
          .replace(/\s/g, 's')
          .replace(/[0-9a-f][1-9a-f]/g, 
            (match) => `<mark style="background-color: #${md5(match).slice(0,6)}80">${match}</mark>`);
        
        
        return text;
      },

      handleInput() {
        const highlightedText = this.applyHighlights(this.sequence);
        this.highlights = highlightedText;
      },

      handleScroll(e) {
        const {
          scrollTop,
          scrollLeft
        } = e.target;

        this.$refs.highlights.scrollTo(scrollLeft, scrollTop);
        this.$refs.background.scrollTo(scrollLeft, scrollTop);
      },

      toHex(i){
        const hex = (i+1).toString(16);
        return hex.length == 1 ? '0' + hex : hex;
      },
      instrTemplate(name){
        return {
          name,
          "osc": "sin",
          "gain": 1,
          "note": "E6",
          "attack": "1000",
          "decay": "1",
          "sustain": -20,
          "release": 1,
          "filterType": "bpf",
          "cutoff": 2000,
          "resonance": 30,
          "time": 1500
        };
      },
      addInstrument(){
        this.instruments.push(this.instrTemplate('instr_' + (this.instruments.length+1)))
      },

      async buildSong(){
        try{
          const synth = synthkit.createSynth();

          const sequence = [];
          const rows = this.sequence.split('\n');

          const orkestar = {};

          for(let k = 0; k < rows.length; k++){
            sequence.push([]);
            const row = rows[k];
            for(let i = 0; i < row.length && row[i+1] !== undefined; i += 2){
              const id = row[i] + row[i+1];

              const instr = this.instruments.find((_, j) => this.toHex(j) == id);
              sequence[k].push(instr ? id : null);

              if(!orkestar[id]){
                if(instr)
                  orkestar[id] = this.createOsc(synth, instr, id);
              }
            }
          }

          const {
            tempo, beat
          } = this.settings;

          const seq = synth.clock(beat, tempo);

          let tick = 0;
          seq.ontrigger = () => {
            tick = tick % sequence[0].length;

            for(let k = 0; k < sequence.length; k++){
              const instr_id = sequence[k];
              const prev = instr_id[tick-1];
              const cur = instr_id[tick];

              if(prev){
                // console.log(orkestar[prev]);
                orkestar[prev].envelope.off();
              }

              if(cur){
                // console.log(orkestar[cur]);
                orkestar[cur].envelope.on();
              }
            }

            tick++;
          }

          const mixer = synth.mixer();
          mixer.inputs = Object.values(orkestar).map(i => i.osc.output)

          const iterations = ~~(44100 / (tempo / 60 * beat / 4)) * sequence[0].length;

          return await this.render(synth.delta, mixer.output, iterations);
        }catch(e){
          console.log(e);
          this.status = e.message;
        }
      },

      concat(_a, i, a = []){
        return i > 0 ? this.concat(_a, --i, a.concat(_a)) : a;
      },

      async writeWav(){
        try{
          const song = await this.buildSong();

          this.status = 'writing...';
          const buffer = await wavencoder.encode({
            sampleRate: 44100,
            channelData: [
              new Float32Array(this.concat(song, this.loopsCount)),
              new Float32Array(this.concat(song, this.loopsCount))
            ]
          })

          this.status = 'ready.';
          saveAs(new Blob([buffer]), 'song.wav');
        }catch(e){
          console.log(e);
          this.status = e.message;
        }
      },

      async play(){
        try{
          this.isPlaying = true;
          this.createContext();

          const fakesynth = {
            delta: () => {}
          };

          const song = await this.buildSong();

          setTimeout(() => {
            this.status = 'playing';
            let i = -1;
            this.synthNode = synthkit.createSynthNode(this.audioCtx, fakesynth, () => song[++i % song.length]);
            this.synthNode.connect(this.gainNode);
          }, 200);
        }catch(e){
          console.log(e);
          this.status = e.message;
        }
      },

      render(delta, output, iterations){
        this.status = 'rendering...';
        return new Promise((res, rej) => {
          try{
            const song = [];
            setImmediate(() => {
              const song = [];
              for(let i = 0; i < iterations; i++){
                delta();
                song.push(output());
              }

              this.status = 'ready.';
              res(song);
            });
          }catch(e){
            rej(e);
          }
        })
      },

      stop(){
        this.status = 'ready.';
        this.isPlaying = false;
        if(!!this.synthNode)
          this.synthNode.disconnect(this.gainNode);
      },

      createOsc(synth, opts, id = '00'){
        const envelope = synth.eg();

        envelope.attack = () => 1/(+opts.attack < 0.001 ? 0.001 : opts.attack);
        envelope.decay = () => 1/(+opts.decay < 0.001 ? 0.001 : opts.decay);
        envelope.sustain = () => +opts.sustain;
        envelope.release = () => +1/(+opts.release < 0.001 ? 0.001 : opts.release);

        const osc = synth.osc();
        osc.type = () => opts.osc;
        osc.freq = () => this.getFreq(opts.note);
        osc.gain = () => envelope.output() * opts.gain;

        if(Object.values(synthkit.FilterType).indexOf(opts.filterType) != -1){
          const filter = synth.filter(opts.filterType, opts.cutoff, opts.resonance);
          filter.input = osc.output;
          return { envelope, osc: filter, id };
        }else{
          return { envelope, osc, id };
        }
      },

      createContext(){
        if(!this.audioCtx){
          this.audioCtx = new AudioContext();
          this.gainNode = this.audioCtx.createGain();
          this.gainNode.gain.value = 0.2;
          this.gainNode.connect(this.audioCtx.destination);
        }
      },

      test(opts){
        try{
          this.createContext();

          const synth = synthkit.createSynth();
          const mixer = synth.mixer();

          const { osc, envelope } = this.createOsc(synth, opts);
          envelope.on();

          mixer.inputs.push(osc.output);
          const synthNode = synthkit.createSynthNode(this.audioCtx, synth, mixer.output);

          synthNode.connect(this.gainNode);
          setTimeout(() => {
            envelope.off();
            envelope.onstop = () => synthNode.disconnect(this.gainNode)
          }, +opts.time);
        }catch(e){
          console.log(e);
          this.status = e.message;
        }
      },

      getFreq(note){
        const reg = /([CDEFGAB])([#b]*)([1-8])/i;
        const match = note.match(reg);
        if(match == null) throw 'wrong note';

        const [ full, letter, accidental, octave ] = note.match(reg);

        const fun = n => 440 * Math.pow(2, (n - 69)/12);
        const sharp = ['C', 'C#', 'D', 'D#', 'E', 'F', 'F#', 'G', 'G#', 'A', 'A#', 'B' ];
        const flat = ['C', 'Db', 'D', 'Eb', 'E', 'F', 'Gb', 'G', 'Ab', 'A', 'Bb', 'B' ];
        let index = -1;

        const _note = letter.toUpperCase() + accidental;
        if(accidental == '#'){
          index = sharp.indexOf(_note);
        }else{
          index = flat.indexOf(_note);
        }
        
        const n = (+octave)*12 + index;
        return fun(n);
      }
    }
  }
</script>