<template>
<div>
        <video id="video" style="width:100%;" autoplay></video> </br>
		<button @click="snap" class='btn btn-success form-control'>Scan</button><hr>

		<img :src="photo"> </br>
		</br><button v-on:click="processImport" class='btn btn-primary form-control'>Save capture file</button>
</div>				


         

</template>

<script src="https://webrtc.github.io/adapter/adapter-latest.js"></script>

<script>
export default {
	data()
	{
		return {
			photo: null,
			filename:"",
			serialcode:Math.random().toString(36).substring(7),

        };
	},
	mounted()
	{
		console.log(document.getElementById("video"));
		
		navigator.mediaDevices.enumerateDevices().then( devices =>
		{
			devices= devices.filter( v => (v.kind=="videoinput"));
			console.log("Found "+devices.length +" video devices");
			let lastDevice= devices[devices.length-1];
			devices= devices.filter( v => (v.label.indexOf("back")>0));
			let device= null;
			if( devices.length > 0 )
			{
				console.log("Taking a 'back' camera");
				device= devices[devices.length-1];
			}
			else
			{
				console.log("Taking last camera");
				device= lastDevice;
			}
				
			if( !device )
			{
				console.log("No devices!");
				return;
			}
			
			let constraints =
			{
				audio: false, 
				video: {
					deviceId: { ideal: device.deviceId },
					width: { ideal: window.innerWidth },
					height: { ideal: window.innerHeight }
				}
			};
			navigator.mediaDevices.getUserMedia(constraints)
			.then( stream =>
			{
				try {
				  document.getElementById("video").srcObject = stream;
				} catch (error) {
				  document.getElementById("video").srcObject = URL.createObjectURL(stream);
				}
				//info.innerHTML+= "<pre>DONE</pre>";
				console.log("DONE");
			})
			.catch( err =>
			{
				console.log(err.name + ": " + err.message);
			});	
		})
		.catch( err =>
		{
			console.log(err.name + ": " + err.message);
		});
		
                axios.get(process.env.MIX_APP_URL + '/documents/getroles')
                .then(response =>  {
                  this.access = response.data;
                    console.log(response.data)
                })
                .catch(e => {
                    console.log(e);
                }); 


    },
    methods:
    {
        snap2() {
            console.log(document.getElementById("video"));

        },
        snap () {
            console.log(this.getCanvas());
            const canvas = this.getCanvas();
            this.photo = canvas.toDataURL('image/jpeg');
        },
        getCanvas () {
            
            const video = document.getElementById("video");
            if (!this.ctx) {
                const canvas = document.createElement('canvas');
                canvas.height = video.clientHeight;
                canvas.width = video.clientWidth;
                this.canvas = canvas;
                
                this.ctx = canvas.getContext('2d');
                
                /*if (this.mirror) {
                const context = canvas.getContext('2d');
                context.translate(canvas.width, 0);
                context.scale(-1, 1);
                this.ctx = context;
                } else {
                this.ctx = canvas.getContext('2d');
                }*/
            }
            
            const { ctx, canvas } = this;
            ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
            
            return canvas;
			},
		dataURLtoFile(dataurl, filename) {
			var arr = dataurl.split(','), mime = arr[0].match(/:(.*?);/)[1],
				bstr = atob(arr[1]), n = bstr.length, u8arr = new Uint8Array(n);
				while(n--){
					u8arr[n] = bstr.charCodeAt(n);
				}
				return new File([u8arr], filename, {type:mime});
			},
		processImport() {
		
				let data = new FormData();
				var file = this.dataURLtoFile(this.photo, 'jd.jpg');
					// data.append('data', file);
                    // data.append('serialcode', this.serialcode);
                    

                    alert(file);



              			


			
        },
            
    }
    
}
</script>