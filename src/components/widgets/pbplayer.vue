<template>
<div class="h5videowrapper h5container" >
    <video class="h5video" :id="videoid" autoplay webkit-playsinline playsinline @click="loseVideo($event)"></video>

    <div class="h5controls"  style="display:none padding:0px">
        <button type="button" class="btn vidbuttion pull-right" @click="CloseVideo($event)"> <i class="mdi mdi-close"></i></button>
        <button type="button" class="btn vidbuttion pull-right" @click="FullScreen($event)"> <i class="mdi mdi-fullscreen"></i></button>
        <button type="button" class="btn vidbuttion pull-right" @click="PtzControlShow($event)"> <i class="mdi mdi-parking"></i></button>
        <button type="button" class="btn vidbuttion pull-right rtcbutton" > <i class="mdi mdi-format-title"></i></button>
        <button type="button" class="btn vidbuttion pull-right" @click="DoManualRecordStop($event)"> <i class="mdi mdi-stop"></i></button>
        <button type="button" class="btn vidbuttion pull-right" @click="DoManualRecordStart($event)"> <i class="mdi mdi-record"></i></button>
        <button type="button" class="btn vidbuttion pull-right" @click="DoSnapshot($event)"> <i class="mdi mdi-camera"></i></button>
        <button type="button" class="btn vidbuttion pull-right" @click="DoSnapshotWeb($event)"> <i class="mdi mdi-file-image"></i></button>
        <!-- audio
        <button type="button" class="btn vidbuttion pull-right" > <i class="mdi  mdi-record"></i></button>
        <button type="button" class="btn vidbuttion pull-right" href="#"> <i class="mdi mdi-volume-high"></i></button>
        <button type="button" class="btn vidbuttion pull-right" href="#"> <i class="mdi mdi-volume-off"></i></button>
        -->
    </div>

    <div class="ptzcontrols"  style="display:none padding:0px">
        <div class="row ">
        <button type="button" class="btn ptzbuttonnone pull-right" href="#"> <i ></i></button>
        <button type="button" class="btn ptzbutton pull-right" href="#"
            @mousedown ="PtzActionZoomIn($event)" @mouseup="PtzActionStop($event)" @touchstart ="PtzActionZoomIn($event)" @touchend="PtzActionStop($event)"> <i class="mdi  mdi-plus-circle-outline"></i></button>
        <button type="button" class="btn ptzbuttonnone pull-right" href="#"> <i ></i></button>
        <button type="button" class="btn ptzbutton pull-right" href="#"
            @mousedown ="PtzActionUp($event)" @mouseup="PtzActionStop($event)" @touchstart ="PtzActionUp($event)" @touchend="PtzActionStop($event)"> <i class="mdi  mdi-arrow-up"></i></button>
        </div>
        <div class="row ">
        <button type="button" class="btn ptzbuttonnone pull-right" href="#"> <i ></i></button>
        <button type="button" class="btn ptzbuttonnone pull-right" href="#"> <i ></i></button>
        <button type="button" class="btn ptzbutton pull-right" href="#"
            @mousedown ="PtzActionRight($event)" @mouseup="PtzActionStop($event)" @touchstart ="PtzActionRight($event)" @touchend="PtzActionStop($event)"> <i class="mdi  mdi-arrow-right"></i></button>
        <button type="button" class="btn ptzbuttonnone pull-right" href="#"> <i ></i></button>
        <button type="button" class="btn ptzbutton pull-right" href="#"
            @mousedown ="PtzActionLeft($event)" @mouseup="PtzActionStop($event)" @touchstart ="PtzActionLeft($event)" @touchend="PtzActionStop($event)"> <i class="mdi  mdi-arrow-left"></i></button>
        </div>
        <div class="row ">
        <button type="button" class="btn ptzbuttonnone pull-right" href="#"> <i ></i></button>
        <button type="button" class="btn ptzbutton pull-right" href="#"
            @mousedown ="PtzActionZoomOut($event)" @mouseup="PtzActionStop($event)" @touchstart ="PtzActionZoomOut($event)" @touchend="PtzActionStop($event)"> <i class="mdi  mdi-minus-circle-outline"></i></button>
        <button type="button" class="btn ptzbuttonnone pull-right" href="#"> <i ></i></button>
        <button type="button" class="btn ptzbutton pull-right" href="#"
            @mousedown ="PtzActionDown($event)" @mouseup="PtzActionStop($event)" @touchstart ="PtzActionDown($event)" @touchend="PtzActionStop($event)"> <i class="mdi  mdi-arrow-down"></i></button>
        </div>
    </div>
</div>
</template>

<script>
import '../../assets/adapter.js'
import {H5sPlayerWS,H5sPlayerHls,H5sPlayerRTC} from '../../assets/h5splayer.js'
import {H5siOS,H5sPlayerCreate} from '../../assets/h5splayerhelper.js'
export default {
    name: 'pbplayer',
    props:['h5id', 'h5videoid','xzvalue',"Adswitch"],
    data () {
        return {
            videoid: this.h5videoid,
            h5handler: undefined,
            currtoken: undefined,
            ptzshow: false,
            proto: 'WS',
            tokenplay:"",
        }
    },
    activated() {
        console.log(this.h5id, "activated");
    },
    deactivated() {
        console.log(this.h5id, "deactivated");
    },
    beforeDestroy() {
        //console.log(this.h5id, "beforeDestroy");
        if (this.h5handler != undefined)
        {
            this.h5handler.disconnect();
            delete this.h5handler;
            this.h5handler = undefined;
        }
        this.currtoken = undefined;
    },
    destroyed() {
        //console.log(this.h5id, "destroyed");
    },
    mounted() {
        console.log(this.$store.state.xzvalue)
        var h5videoid1=this.h5videoid
        console.log(this.h5videoid);
        this.$emit("h5videoid1",h5videoid1);

        var $container = $("#"+this.h5id);
        var $video =$container.children("video");
        var videodom = $container.children("video").get(0);
        var $controls = $container.children(".h5controls");
        var $rtcbutton = $controls.children(".rtcbutton");

        let _this = this;
        this.$root.bus.$on('pbplayer', function(token, id)
        {
            if (_this.h5id != id)
            {
                return;
            }
            _this.PlayVideo(token);
            _this.tokenplay=token;
        });

        this.$root.bus.$on('liveplayproto', function(proto)
        {
            _this.proto = proto;
            //console.log("liveplayproto", _this.proto);
        });

        // control visibility
        $container.on("mouseover mouseout touchstart touchmove", function(e) {
            $controls.css("display", e.type === "mouseout" ? "none" : "block");
            //$controls.css("display", e.type === "touchend" ? "none" : "block");
        });
    },
    
    methods: {

        PlaybackCB(event, userdata)
        {
            //console.log("Playback callback ", event);
            var msgevent = JSON.parse(event);
            if (msgevent.type === 'H5S_EVENT_PB_TIME')
            {
                userdata.msg(msgevent.pbTime.strTime);
            }
            
        },
        msg(pbtime){
            //this.$emit('childByValue', this.name);
            this.$emit("titleChanged",pbtime);
            //console.log("1111",pbtime);
            
            
        },
		PlayVideo(token)
        {
            var timevalue=this.$store.state.xzvalue;
            //var timevalue=this.xzvalue;
            console.log(timevalue);
            
            var year = timevalue.getFullYear();
            var month = timevalue.getMonth() + 1;
            var strDate = timevalue.getDate();
            var timevalues=new Date(year+"-"+month+"-"+strDate+" 00:00:00").toISOString();
            var timevaluee=new Date(year+"-"+month+"-"+strDate+" 24:00:00").toISOString();
            // var timevalues=""+year+"-"+month+"-"+strDate+"T00:00:00+08:00";
            // var timevaluee=""+year+"-"+month+"-"+strDate+"T24:00:00+08:00";
            console.log(year,month,strDate);
            console.log(timevalues,timevaluee);
            //return false;
        	if (this.h5handler != undefined)
        	{
        		this.h5handler.disconnect();
        		delete this.h5handler;
        		this.h5handler = undefined;
        	}
        	this.currtoken = token;
        	console.log("play ", token);
        	var root = process.env.API_ROOT;
        	var wsroot = process.env.WS_HOST_ROOT;
        	if (root == undefined){
        		root = window.location.protocol + '//' + window.location.host + window.location.pathname;
        	}
        	if (wsroot == undefined)
        	{
        		wsroot = window.location.host;
        	}
			var pbconf1 = {
				begintime: timevalues,
				endtime: timevaluee,
                showposter:true, //'true' or 'false' show poster
                callback: this.PlaybackCB,
	            serverpb: this.Adswitch, 
				userdata:  this // user data
            };
        	let conf = {
        		videoid: this.h5videoid,
				protocol: window.location.protocol, //http: or https:
				host: wsroot, //localhost:8080
				rootpath:'/', // '/'
				token:token,
				pbconf: pbconf1, //This is optional, if no pbconf, this will be live.
				hlsver:'v1', //v1 is for ts, v2 is for fmp4
				session: this.$store.state.token
        	};
        	var $container = $("#"+this.h5id);
        	var $controls = $container.children(".h5controls");
        	var $rtcbutton = $controls.children(".rtcbutton");

        	if (this.proto == 'RTC' || (H5siOS() === true))
        	{
        		$rtcbutton.css("display", "block");
        		this.h5handler = new H5sPlayerRTC(conf);
        	}else
        	{
        		$rtcbutton.css("display", "none");
        		this.h5handler = new H5sPlayerRTC(conf);
            }
            //console.log("v1",this.h5handler);

        	this.h5handler.connect();
			setTimeout(function(){
				this.h5handler.start();
            }.bind(this),500);
            var timetz=this.h5handler;
            //console.log("000",timetz);
            this.$emit("vv",timetz);
            

        },
        loseVideo(){
            var h5handler=this.currtoken;
            console.log("v3",h5handler);
            this.$emit("videh5handler",h5handler);
        },
		//以上不知道
        FullScreen(event)
        {
            var elem = $("#"+this.h5id).get(0);
            //var elem = $("#videoPanel");
            console.log('panelFullScreen', event);
            if (
            document.fullscreenEnabled ||
            document.webkitFullscreenEnabled ||
            document.mozFullScreenEnabled ||
            document.msFullscreenEnabled
            ) {
            if (
                document.fullscreenElement ||
                document.webkitFullscreenElement ||
                document.mozFullScreenElement ||
                document.msFullscreenElement
            ) {
                if (document.exitFullscreen) {
                    document.exitFullscreen();
                } else if (document.webkitExitFullscreen) {
                    document.webkitExitFullscreen();
                } else if (document.mozCancelFullScreen) {
                    document.mozCancelFullScreen();
                } else if (document.msExitFullscreen) {
                    document.msExitFullscreen();
                }
                console.log("========  updateUIExitFullScreen");
                this.updateUIExitFullScreen();
            } else {
                  console.log('panelFullScreen3');

                if (elem.requestFullscreen) {
                    elem.requestFullscreen();
                } else if (elem.webkitRequestFullscreen) {
                    elem.webkitRequestFullscreen();
                } else if (elem.mozRequestFullScreen) {
                    elem.mozRequestFullScreen();
                } else if (elem.msRequestFullscreen) {
                    elem.msRequestFullscreen();
                }
            }
        } else {
            console.log('Fullscreen is not supported on your browser.');
        }
        },
        PtzControlShow(event)
        {
            console.log(this.ptzshow);
            var $container = $("#"+this.h5id);
            var $controls = $container.children(".ptzcontrols");
            if (this.ptzshow == false)
            {
                $controls.css("display", "block");
                this.ptzshow = true;
            }else
            {
                $controls.css("display", "none");
                this.ptzshow = false;
            }
        },

        PtzActionZoomIn(event)
        {
            console.log("PtzActionZoomIn");
            this.PtzAction('zoomin');
        },
        PtzActionZoomOut(event)
        {
            this.PtzAction('zoomout');
        },
        PtzActionLeft(event)
        {
            this.PtzAction('left');
        },
        PtzActionRight(event)
        {
            this.PtzAction('right');
        },
        PtzActionUp(event)
        {
            this.PtzAction('up');
        },
        PtzActionDown(event)
        {
            this.PtzAction('down');
        },
        PtzActionStop(event)
        {
            console.log("PtzActionStop");
            this.PtzAction('stop');
        },

        PtzAction(action)
        {
            if (this.h5handler == undefined)
            {
                return;
            }
            let _this =this;
            var root = process.env.API_ROOT;
            if (root == undefined){
                root = window.location.protocol + '//' + window.location.host + window.location.pathname;
            }

            var ptzcmd = "token=" + this.currtoken + "&action=" + action + "&speed=0.5";
            console.log("ptzcmd", ptzcmd);

            var url = root + "/api/v1/Ptz?" + ptzcmd  + "&session="+ this.$store.state.token;

            this.$http.get(url).then(result => {
                console.log(result);
                if (result.status == 200)
                {

                }
            }).catch(error => {
                console.log('ptz failed!', error);
            });
        },
        DoManualRecordStart(event)
        {
            if (this.h5handler == undefined)
            {
                return;
            }
            let _this =this;
            var root = process.env.API_ROOT;
            if (root == undefined){
                root = window.location.protocol + '//' + window.location.host + window.location.pathname;
            }

            var record = "token=" + this.currtoken + "&duration=300";
            console.log("record cmd", record);

            var url = root + "/api/v1/ManualRecordStart?" + record  + "&session="+ this.$store.state.token;

            this.$http.get(url).then(result => {
                console.log(result);
                if (result.status == 200)
                {
                    this.$Notice.info({
                        title: "Manual Start Record successfully"
                    })
                }
            }).catch(error => {
                console.log('Record failed!', error);
                this.$Notice.info({
                    title: "Record failed !"
                })
            });
        },
        DoManualRecordStop(event)
        {
            if (this.h5handler == undefined)
            {
                return;
            }
            let _this =this;
            var root = process.env.API_ROOT;
            if (root == undefined){
                root = window.location.protocol + '//' + window.location.host + window.location.pathname;
            }

            var record = "token=" + this.currtoken + "&duration=300";
            console.log("record cmd", record);

            var url = root + "/api/v1/ManualRecordStop?" + record  + "&session="+ this.$store.state.token;

            this.$http.get(url).then(result => {
                console.log(result);
                if (result.status == 200)
                {
                    this.$Notice.info({
                        title: "Manual Stop Record successfully"
                    })
                }
            }).catch(error => {
                console.log('Record failed!', error);
                this.$Notice.info({
                    title: "Record failed !"
                })
            });
        },
        DoSnapshot(event)
        {
            if (this.h5handler == undefined)
            {
                return;
            }
            let _this =this;
            var root = process.env.API_ROOT;
            if (root == undefined){
                root = window.location.protocol + '//' + window.location.host + window.location.pathname;
            }

            var snapshot = "token=" + this.currtoken;
            console.log("snapshot cmd", snapshot);

            var url = root + "/api/v1/Snapshot?" + snapshot  + "&session="+ this.$store.state.token;

            this.$http.get(url).then(result => {
                console.log(result);
                if (result.status == 200)
                {
                    this.$Notice.info({
                        title: "Snapshot successfully"
                    })
                }
            }).catch(error => {
                console.log('Snapshot failed!', error);
                this.$Notice.info({
                    title: "Snapshot failed !"
                })
            });
        },
        DoSnapshotWeb(event)
        {
            var fileName = '1';
            const date = new Date();
            fileName = this.currtoken + '_' + date.getFullYear() + '-' + (date.getMonth() + 1)
                         + '-' + date.getDate() + '-' + date.getHours() + '-' + date.getMinutes() + '-' + date.getSeconds();
            var video = $("#" + this.h5videoid).get(0);
            var w = video.videoWidth;//video.videoWidth * scaleFactor;
            var h = video.videoHeight;//video.videoHeight * scaleFactor;
            var canvas = document.createElement('canvas');
            canvas.width = w;
            canvas.height = h;
            var ctx = canvas.getContext('2d');
            ctx.drawImage(video, 0, 0, w, h);
            var MIME_TYPE = "image/png";
            var imgURL = canvas.toDataURL(MIME_TYPE);

            var dlLink = document.createElement('a');
            dlLink.download = fileName;
            dlLink.href = imgURL;
            dlLink.dataset.downloadurl = [MIME_TYPE, dlLink.download, dlLink.href].join(':');

            document.body.appendChild(dlLink);
            dlLink.click();
            document.body.removeChild(dlLink);
        }
    }
}
//fill scale-down
</script>

<style scoped>

.h5video{
   object-fit: fill;
}

.h5videowrapper{
    padding: 0px;
    height: 100%;
    width: 100%;
}

video {
    width: 100%;
    height: 100%
}

.vidbuttion {
    height: 24px;
    width: 24px;
    padding:0px;
    margin: 0px;
    margin-left: 5px;
    opacity: 0.60;

}

.vidbuttion:hover {
    /*background-color: #3c8dbc;*/
    cursor: pointer;
    color: rgb(187, 184, 184);
}

.ptzbutton {
    height: 24px;
    width: 24px;
    padding:0px;
    margin: 0px;
    opacity: 1;
    background:rgba(255,255,255,0.3);
}

.ptzbuttonnone {
    height: 24px;
    width: 24px;
    padding:0px;
    margin: 0px;
    margin-right: 0px;
    opacity: 0;
    background:rgba(255,255,255,0);
}

.ptzbutton:hover {
    /*background-color: #3c8dbc;*/
    cursor: pointer;
    color: rgb(187, 184, 184);
}

.h5container {
    position:relative;
    display:inline-block;
}

.rtcbutton {
    display:none;
}

.h5container > .h5controls {
    position:absolute;
    top:0;
    background:rgba(255,255,255,0.3);
    padding:0px;
    box-sizing:content-box;
    z-index:10000;
    width: 100%;
    display:none;
}
.h5container > .ptzcontrols {
    position:absolute;
    bottom:0;
    background:rgba(255,255,255,0);
    padding:0px;
    box-sizing:content-box;
    z-index:11000;
    width: 100%;
    display:none;
}

</style>
