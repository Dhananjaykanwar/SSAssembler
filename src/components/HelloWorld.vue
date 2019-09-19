<template>
<center class="mainPage">
  <v-container style="max-width:90%">
    <v-layout>
      <v-flex xs6>
      <v-layout column>
      <v-flex xs12  class="codeContainer">
      <v-flex class="labelForCode">Code</v-flex>
      <v-flex class="codeOutline">
        <textarea label="Code" id="codeArea" v-model="instructions" @read:separate="readSeparate"></textarea>
      </v-flex>
      </v-flex>
      <v-flex xs12  class="codeContainer">
      <v-flex class="labelForCode" mt-4>Symbol Table</v-flex>
      <v-flex>
        <v-flex xs12 class="symbolTable" v-model="outputData">
          <table width="100%" style="text-align:left">
            <tr>
              <th>Symbol</th>
              <th>Location</th>
            </tr>
            <tr v-for="i in symbolTable" :key=i><td class="tableItem">{{ i }}</td>
            <td class="tableItem" style="border-right:none;">unassigned</td>
            </tr>
          </table>
        </v-flex>
      </v-flex>
      </v-flex>
      </v-layout>
      </v-flex>
      <v-flex xs4 style="height:500px">
        <table style="height:fill-height;width:100%">
          <tr>
            <td>
            <v-flex class="labelForCode">Output</v-flex>
            <v-flex xs12 class="outputScreen" v-model="outputData"></v-flex>
            </td>
          </tr>
          <tr>
            <td>
              <v-flex class="labelForCode">Cpu &amp; Memory</v-flex>
              <v-flex xs12 class="ramContainer">
              <div class="RAM">
                <v-flex xs12 style="height:80px">
                  <h5>Registers/Flags</h5>
                  <v-simple-table style="width:100%;border-bottom:2px solid #ddd;">
                    <tr>
                      <td v-for="y in 8" :key=y class="registers">{{ y }}</td>
                    </tr>
                  </v-simple-table>
                </v-flex>
                <h5>RAM</h5>
                <table style="height:fill-height;width:100%" class="tbl">
                  <tr v-for="x in 16" :key=x id="used"><td v-for="i  in 16" :key=i id="ramItem" :style="{color:InstructStore[(16*(x-1)+i-1)]}">00</td></tr>
                </table>
              </div>
              </v-flex></td>
               </tr>
               <tr>
                 <td><v-flex mt-4 class="labelForCode">Literal Table</v-flex>
                <v-flex xs12 class="outputScreen" v-model="outputData">
            <table width="100%" style="text-align:left;padding:3%">
            <tr>
              <th>Symbol Type</th>
              <th>Symbol Value</th>
              <th>Memory Address</th>
            </tr>
                <tr v-for="i in literalTable" :key=i><td class="tableItem">{{ i.type }}</td>
                <td class="tableItem" style="border-right:none;">{{ i.value }}</td>
                <td class="tableItem" style="border-right:none;">{{ i.memAdr }}</td>
            </tr>
          </table> 
                  </v-flex></td>
               </tr>
               <tr>
                 <td><v-flex class="labelForCode">Machine-Op Table</v-flex>
                <v-flex xs12 class="outputScreen" v-model="outputData">
                  <table width="100%" style="text-align:left;padding:3%">
            <tr>
              <th>Mnemonic Op-code</th>
              <th>Binary Op-code</th>
              <th>Instruction Length(bits)</th>
            </tr>
            <tr v-for="(i,index) in motTable[0].mcode" :key=i><td class="tableItem">{{ i }}</td>
            <td class="tableItem">{{ motTable[0].bcode[index] }}</td>
            <td class="tableItem" style="border-right:none;">32</td>
            </tr>
          </table>  
                </v-flex></td>
               </tr>
        </table>
      </v-flex>
    </v-layout>
  </v-container>
</center>
</template>

<script>

export default {
  name: 'HelloWorld',
  data(){
        return{
          instructions : [],
          locationCounter : 0,
          outputData: '',
          potTable:[
            "DS","DC","USING","DROP","END","START"
          ],
          motTable:[
            {
              mcode : ['L','A','ST'],
              bcode : ['1F','1A','1C']
            }
          ],
          symbolTable : [],
          memAdr: [],
          literalTable: [],
          InstructStore : [],
        }
  },
  methods : {
    readSeparate(){
      let ins = this.instructions.split(';').filter((elem)=>{
          return /\S/.test(elem);
      });
      let codeToProcess = ins.map((elem) => {
        if (elem === 'START') return elem; 
        return elem.slice(1,elem.length);
        });
      this.processInstructions(codeToProcess);
      this.storeInstruction(this.instructions.split(/\n/));
    },
    processInstructions(cTP){
        let Used = (cTP.map((elem) => { 
          if(elem === 'START') return " ";
          else
          {
          let a=elem.split(" "); 
          return a[0];
          }
          })).join(" ").trim();
          let elementArray = Used.split(/\s|\n/g),cE='';
          let literal=[];
          let elementLiterals = (cTP.map((elem) =>
          {           
          if(elem === 'START') return " "; 
          return elem; 
          })).join("-").trim();
          if(cTP[this.locationCounter].toUpperCase()==='START')
          while(cE!=='END')
          {
          cE=elementArray[this.locationCounter];
          if([...cE].indexOf(':')!== -1){
            cE=cE.slice(0,cE.length-1);
            this.symbolTable.push(cE);
            this.locationCounter++;
          }
          if(this.potTable.indexOf(cE)!==-1)
          {
            if(cE === 'DS' || cE === 'DC')
            {
                literal = elementLiterals.split("-").filter(elem => elem.split(" ").indexOf(cE) !== -1).join(" ");
                literal = literal.split(' ');
                literal = literal[1].split('*');
                let obj={
                  type:literal[0],
                  value:literal[1],
                  memAdr:this.memAdr[this.memAdr.length-1]
                };
                console.log(obj);
                this.literalTable.push(obj);

                this.locationCounter++;
            }
          }
          else
          if(this.motTable[0].mcode.indexOf(cE)!==-1)
          {
            this.locationCounter++;
          }
        }
    },
    storeInstruction(cTP){
      this.InstructStore = [];
      for(let i=0;i<cTP.length;i++){
          this.InstructStore.push('red');
      }
      this.ramAssigner(this.InstructStore,cTP);
    },
    ramAssigner(rG,cTP){
      let registered=0;
      $(document).ready(function(){
      for(let i=0;i<=rG.length/16;i++)
      for(let j=0;j<=rG.length%16;j++)
      {
        $("tr:nth-child(1)>#ramItem:nth-child("+registered+")").text("R"+registered);
        registered++;
      }
      });
      let mAE = [...cTP].filter(elem => elem.match(/^DS|DC/i));
      let exclude = (n) =>{
        if(n<=this.InstructStore.length || memAdr.indexOf(n) !== -1)
        return false;
        return true;
      }
      async function randomExcluded(min=0, max=256) {
        let n = await Math.floor(Math.random() * (max-min) + min);
        console.log(n);
        if (exclude(n)) return randomExcluded(0,256);
        return n;
        }   
      let randGen = randomExcluded
                    .then(1elem => return elem)
                    .catch(err => err);
      console.log(randGen);

    }

  }
};
</script>
<style>
.RAM {
  height: 100%;
  width:100%;
  padding: 2%;
}
.ramContainer
{
  padding:4%;
  height:418px;
  border:1px solid #ddd;
  border-bottom-right-radius: 5px;
  border-bottom-left-radius: 5px;
}
#ramItem
{
  font-size: x-small;
  font-family: 'montserrat',sans-serif;
  font-weight: bold;   
  color : #428bca; 
}
.codeOutline
{
 padding-top: 1%;
 border:1px solid #ddd;
 border-bottom-right-radius: 5px;
 border-bottom-left-radius: 5px;
 height:550px;
}
.registers
{
  font-size:10px;
  font-family: 'Montserrat', sans-serif;
  color:#428bca;
  font-weight: bold;
  height:30px !important;
  padding:0 4px !important;
  text-align: center;
}
.mainPage
{
  background-color:#fff;
}
#codeArea
{
  height: 520px;
  width:510px;
  border: 1px solid #ddd;
}
.labelForCode
{
  border-top-right-radius: 5px;
  border-top-left-radius: 5px;
  height:35px;
  border: 1px solid #ddd;
  border-bottom-color: #fff;
  padding:1%;
  font-family: 'Permanent Marker', cursive;
  text-align: left;
  background: #f5f5f5;
}
.codeContainer
{
  height:600px;
  margin-right:3%;
}
.outputScreen
{
  height:auto;
  min-height: 80px;
  width:470px;
  border:1px solid #ddd;
  margin-bottom:3%;
  border-bottom-right-radius: 5px;
  border-bottom-left-radius: 5px; 
}
.symbolTable{
  height: auto;
  min-height: 213px;
  border:1px solid #ddd;
  margin-bottom:3%;
  border-bottom-right-radius: 5px;
  border-bottom-left-radius: 5px;
  padding: 3%;
}
h5{
  color:#777;
  font-weight:400;
  font-size: 13px;
}
.tableItem{
  font-size:14px;
  text-transform: uppercase;
  font-family: 'Montserrat', sans-serif;
  font-weight: bold;
  height:30px !important;
  padding:0 4px !important;
  border-bottom: 1px solid #ddd;
  border-right: 1px solid #ddd;
  text-align: center;
}
th{
  background-color:lightblue;
  padding: 0 4px;
  border-bottom: 1px solid #fff;
  text-align: center;
  font-family: 'Montserrat', sans-serif;
  }
th:nth-child(1){
  border-top-left-radius: 5px;
}
th:nth-last-child(1){
  border-top-right-radius: 5px;
}
</style>
