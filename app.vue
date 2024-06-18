<script>
export default {
	data() {
		return {
			gsheet: {},
			rows: [],
			rooms: new Set(),
			roomData: {},
			room: '',
			baseStation: '',
			remote: ''
		}
	},
	async beforeMount() {
		this.gsheet = await $fetch('https://docs.google.com/spreadsheets/d/1Psj_8F7Sa8cvTxJmyV3bRmMrBp65scP9oVC7AQ5bbeo/gviz/tq?tq&gid=0', {
            parseResponse: (txt) => JSON.parse(txt.replace('/*O_o*/\n', '').replace(/(^google\.visualization\.Query\.setResponse\(|\);$)/g,''))
        })
		for (let s = 0; s < this.gsheet.table.rows.length; s++) {
            let data = {}
            let row = this.gsheet.table.rows[s].c
            for (let c = 0; c < this.gsheet.table.cols.length; c++) {
                if (row[c]) {
                    data[this.gsheet.table.cols[c].label] = row[c].v
                    if (row[c].f) {
                        data[this.gsheet.table.cols[c].label] = row[c].f
                    }
                }
                else {
                    data[this.gsheet.table.cols[c].label] = ''
                }
            }
            this.rows.push(data)
            this.rooms.add(data['Room'])
			this.roomData[data['Room']] = data
        }
	},
	computed: {
		channels() {
			if (this.baseStation == 'Old Base Station' || this.remote == '16 Channel Remote') {
				return 4
			}
			return 8
		},
		baseStationSolution() {
			if (this.channels == 4) {
				if (this.baseStation == 'Old Base Station') {
					return this.roomData[this.room]['Alt DIP']
				}
				else {
					return this.roomData[this.room]['Alt DIP Compat']
				}
			}
			return this.roomData[this.room]['DIP']
		},
		remoteSolution() {
			if (this.channels == 4) {
				if (this.remote == '16 Channel Remote') {
					return this.roomData[this.room]['Alt DIP']
				}
				else {
					return this.roomData[this.room]['Alt DIP Compat']
				}
			}
			return this.roomData[this.room]['DIP']
		}
	},
	methods: {
		reset() {
			this.room = ''
			this.baseStation = '',
			this.remote = ''
		}
	}
}
</script>

<template>
	<Head>
		<Title>CONNECT24 Perfect Cues</Title>
		<Link rel="apple-touch-icon" sizes="76x76" href="/apple-touch-icon.png"></Link>
		<Link rel="icon" type="image/png" sizes="32x32" href="/favicon-32x32.png"></Link>
		<Link rel="icon" type="image/png" sizes="16x16" href="/favicon-16x16.png"></Link>
		<Meta name="theme-color" content="#19369a"></Meta>
	</Head>
<div>
	<div class="header" @click="reset()">
		<div class="logo"><img src="@/assets/cvent-connect-logo.png"></div>
		<div class="dsan"><img src="@/assets/pcue_logo.gif"></div>
	</div>

	<div class="start" v-if="room === ''">
		<div class="bubble">
			<img src="/assets/ce-white.png" />
			<div>Use this applet to set your Perfect Cue to ensure their are no conflicts.</div>
		</div>
		<h1>
			Select an Area:
		</h1>
		<div class="start-btns">
			<button v-for="r in rooms" class="room-btn" @click="room = r">{{ r }}</button>
		</div>
	</div>
	<div class="roomed" v-else>
		<h2>{{ room }}</h2>
		<h3 v-if="baseStation != ''">{{ baseStation }}</h3>
		<h4 v-if="remote != ''">{{ remote }}</h4>

		<div class="question" v-if="baseStation === ''">
			<div class="q">
				Select which model of Base Station your kit contains:
			</div>
			<div class="base-stations">
				<div class="option" @click="baseStation = 'Old Base Station'">
					<p><b>Older Model:</b> Two sets of 4 DIP switches, one for Unit ID and one for Channel</p>
					<div class="dips">
						<div class="dip-header">UNIT ID</div>
						<div class="dip-header">CHANNEL</div>
						<div class="fill"><Dip binary="0011"></Dip></div>
						<div class="fill"><Dip :binary="roomData[room]['Alt DIP']"></Dip></div>
					</div>
					
				</div>
				<div class="option" @click="baseStation = 'New Base Station'">
					<p><b>Newer Model:</b> One set of 8 DIP switches</p>
					<div class="dips">
						<div class="dip-header">CHANNEL</div>
						<div class="fill"><Dip :binary="roomData[room]['DIP']"></Dip></div>
					</div>
					
				</div>
			</div>
			<div class="bubble red">
			<img src="/assets/stop.png" />
			<div>Don't start changing DIP switches yet, the next question could change your settings.</div>
		</div>
		</div>

		<div class="question" v-if="baseStation != '' && remote === ''">
			<div class="q">
				Select which model of remote you have:
			</div>
			<div class="base-stations">
				<div class="option" @click="remote = '16 Channel Remote'">
					<p><b>16 Channel:</b> One set of 4 DIP switches</p>
					<div class="fill"><Dip :binary="roomData[room]['Alt DIP']"></Dip></div>
				</div>
				<div class="option" @click="remote = '256 Channel Remote'">
					<p><b>256 Channel:</b> One set of 8 DIP switches</p>
					<div class="fill"><Dip :binary="roomData[room]['DIP']"></Dip></div>
				</div>
			</div>
		</div>

		<div class="solution" v-if="remote != ''">
			<h1>Result</h1>
			<div class="bubble green">
				<img src="/assets/check.png" />
				<div>Nailed it! Set your Perfect Cue to match these settings.</div>
			</div>
			<div class="base" v-if="baseStation === 'Old Base Station'">
				<h5>Base Station</h5>
				<div class="dips">
					<div class="dip-header">UNIT ID</div>
					<div class="dip-header">CHANNEL</div>
					<div class="fill"><Dip binary="0011"></Dip></div>
					<div class="fill"><Dip :binary="baseStationSolution"></Dip></div>
				</div>
			</div>
			<div class="base" v-else>
				<h5>Base Station</h5>
				<div class="dips">
					<div class="dip-header">CHANNEL</div>
					<div class="fill"><Dip :binary="baseStationSolution"></Dip></div>
				</div>
			</div>

			<div class="rem">
				<h5>Remote</h5>
					<div class="fill"><Dip :binary="remoteSolution"></Dip></div>
			</div>
		</div>
	</div>
</div>
</template>


<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto:ital,wght@0,100;0,300;0,400;0,500;0,700;0,900;1,100;1,300;1,400;1,500;1,700;1,900&display=swap');
@import url('/assets/fonts.css');

html, body {
	font-family: "Roboto", sans-serif;
}

html {
	margin: auto;
	max-width: 440px;
	-webkit-tap-highlight-color: transparent;
	user-select: none;
}

.fill {
    width: 100%;
    display: flex;
    flex-direction: row;
    justify-content: center;
}

.logo > img {
	height: 50px;
}

.header {
	text-align: center;
}

.start-btns {
	display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: center;
    align-items: center;
    gap: 15px;
}

.room-btn {
	background-color: black;
    color: white;
    font-family: "roboto", sans-serif;
    border: 0px solid;
    padding: 8px;
    font-size: 22px;
	cursor: pointer;
}

h1 {
	font-family: 'Roboto', sans-serif;
	font-weight: 800;
	text-align: center;
}

h2 {
	text-align: center;
	font-family: 'BrandonText', sans-serif;
	font-size: 44px;
	margin-top: 12px;
}

.option {
    background-color: black;
    color: white;
    padding: 12px;
    text-align: center;
    font-size: 18px;
	margin-top: 25px;
	cursor: pointer;
}

.q {
	font-size: 28px;
	font-weight: 700;
	font-style: italic;
	margin-bottom: 12px;
	text-align: center;
}

.option > p {
    margin-top: 0px;
}

.dips {
	display: grid;
	grid-template-columns: 1fr 1fr;
	border: 2px solid;
    padding: 6px;
	align-items: center;
	text-align: center;
}

h3 {
    text-align: center;
    margin-top: -32px;
    font-weight: 400;
	font-size: 18px;
}

h4 {
	text-align: center;
    font-weight: 400;
    font-style: italic;
    margin-top: -16px;
    font-size: 16px;
}

h5 {
    font-size: 24px;
    font-weight: 300;
    text-align: center;
    margin-bottom: 0px;
}

.bubble {
    background-color: #ff9700;
    display: grid;
    grid-template-columns: auto 1fr;
    padding: 5px;
    align-items: center;
    color: white;
    margin: 10px;
    gap: 5px;
    border-radius: 10px;
}

.bubble.red {
	background-color: #e53835;
}

.bubble.green {
	background-color: #4bae4f;
}

.bubble > img {
	height: 50px;
}
</style>