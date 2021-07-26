** Docker part 1  
Mục tiêu: Hiểu được image với container, build 1 image với docker từ ứng dụng có sẵn  
**Bất cập khi sử dụng cơ chế triển khai ứng dụng truyền thống:  
1. Work on my machine => Chay được trên máy mình nhưng sang máy khác thì die
2. Chạy nhiều ứng dụng với nhiều yêu cầu phiên bản phần mềm tốn thời gian setup lại môi trường
	=> chạy 1 cái Python 2.7 -> cài python 2.7
		chạy 1 cái python 3.1 -> gỡ python 2.7, cài python 3.1
3. Triển khai ứng dụng cần cài đặt 1 đống service liên quan để run ứng dụng (mysql, nodejs, driver,...)

**Để khác phục 3 nhược điểm trên có thể dùng VM 
=> chiếm tài nguyên cố định, có thể không dùng hết nhưng vẫn thích chiếm
=> nặng vãi nồi, không hiệu quả

** Do đó, dùng docker là hữu hiệu nhất
--------------------------------------------------------
------------------ **BẮT ĐẦU THÔI** --------------------  
**I. Các khái niệm cơ bản trong Docker:  
Trong docker, có 2 khái niệm chính cần chú ý đó là Docker file, Image và container. Container có thể hiểu là một kho
chứa, môi trường để chạy ứng dụng. Còn Image là công thức để tạo ra môi trường đó. Container chính là một
instances của Image. Hiểu đơn giản, mapping với việc nấu ăn thì
*Docker file: công thức để tạo ra Image  
*Image: Là hình ảnh của món ăn  
*Container: Là món ăn được tạo ra.  


Hiểu đơn giản vậy đủ rồi, giờ đến hiểu phức tạp (một số tính chất)
(RẢNH THÌ ĐỌC, KHÔNG RẢNH THÌ THÔI NHÁ!!!)
1. Docker file:  
	- Docker file là 1 file có tên là Dockerfile:)) => Quả định nghĩa ảo thật đấy (Đ hiểu ông nào nghĩ ra quả định nghĩa này, t đi copy thôi :V)
	- Docker file là 1 file gồm các bước để tạo ra Image, trong đó ghi rõ các bước để setup các môi trường như nào, tài cái nào về, chạy như nào
	- Cấu trúc 1 file Dockerfile cơ bản nhé:  
		*FROM <image>:<tag> (image là tên image, tag thường là version của iamge đấy)  

		*RUN <install some dependencies>  

		*CMD <command that is executed on `docker container run`>  
2. Image:
	- Image là một file không thể thay đổi, không thể sửa một image đã tồn tại. Có thể hiểu nó là immutable,
	không thể thay đổi sau khi đã tạo.  
	- List ra tất cả các **image** : * docker image ls (ls thì cứ hiểu là list cho dễ nhớ)   
	
3. Container:  
	- Chứa tất cả các yêu cầu để chạy 1 ứng dụng
	- Bị cô lập với máy host (máy cài docker)
	- List ra tất cả các **container** đang chạy: *docker container ls (nhắc lại ls là list nhá)
		|$ docker container ls							|
        	|CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES	|
	- Thêm -a đi : (docker container ls -a), nó ra tất cả cá container đang tồn tại luônnnn (dù là đã exits )
		 
		$ docker container ls -a																							
		|CONTAINER ID   |IMAGE           |COMMAND      |CREATED          |STATUS                      |PORTS     |NAMES		|
		|---------------|----------------|-------------|-----------------|----------------------------|----------|--------------|
		|b7a53260b513   |hello-world     |"/hello"     |5 minutes ago    |Exited (0) 5 minutes ago    |          |brave_bhabha	|
		|1cd4cb01482d   |hello-world     |"/hello"     |8 minutes ago    |Exited (0) 8 minutes ago    |          |vibrant_bell	|
		
**II. Docker CLI cơ bản (Cái này quan trọng)  
Chúng ta sử dụng command line để tương tác với "Docker Engine" bao gồm CLI, Rest API, docker daemon  
Ví dụ nhá: Khi chạy "*docker container run *" thì sau cái màn hình toàn chữ là chữ mình nhìn thấy thì 
Docker sẽ send một request thông qua REST API tới docker daemon để lấy image, container và các tài nguyên khác.

**Danh sách command cơ bản:
1. **docker image ls** => List all image
2. **docker image rm <image>** Xoá image có tên là image
3. **docker image pull <image>** Pull image từ docker registry (Kho chứa image của thằng docker)
4. **docker container ls -a** List all container
5. **docker container run <image>** Run 1 container từ cái image <image>
6. **docker container rm <container>** Remove container có tên <container>
7. **docker container stop <container>** Dừng container có tên <container>
8. **docker container exec <container>** Thực thi 1 command trong container (Ví dụ container <container> đang chạy là ubuntu thì
											*docker exec ubuntu bash* tương ứng với mở terminal của ubuntu ra)

**Có các trường hợp thực hiện lệnh *docker iamge rm hello-world* sẽ không được vì các trường hợp sau:
 => Đã tồn tại một container tương tự đang chạy:
	|$ docker image rm hello-world 
	|Error response from daemon: conflict: unable to remove repository reference 
	|	"hello-world" (must force) - container <container ID> is using its referenced image <image ID>
	Chạy *docker container ls -a* sẽ thấy 
	|$ docker container ls -a
	|CONTAINER ID   IMAGE           COMMAND        CREATED          STATUS                      PORTS     NAMES
	|b7a53260b513   hello-world     "/hello"       35 minutes ago   Exited (0) 35 minutes ago             brave_bhabha
	|1cd4cb01482d   hello-world     "/hello"       41 minutes ago   Exited (0) 41 minutes ago             vibrant_bell
**Có thể filter docker theo tên bằng cách:** *docker container ls -a | grep <container_name>
9. **Docker system prune** dùng để xoá các vùng dữ liệu rác sinh ra khi dừng chạy container (tương tự với image)
10. **Docker container run -d <container_name>** dùng để detect các container so tên là <container_name> đang chạy

**III. Một số shorthand (Câu lệnh ngắn trong docker)


|command							|		explain								|	shorthand
|-----------------------------------|-------------------------------------------|------------------
|docker image ls					|	Lists all images						|	docker images
|docker image rm <image>			|	Removes an image						|	docker rmi
|docker image pull <image>			|	Pulls image from a docker registry		|	docker pull
|docker container ls -a				|	Lists all containers					|	docker ps -a
|docker container run <image>		|	Runs a container from an image			|	docker run
|docker container rm <container>	|	Removes a container						|	docker rm
|docker container stop <container>	|	Stops a container						|	docker stop
|docker container exec <container>	|	Executes a command inside the container |	docker exec

**IV. Running and stopping container

Bắt đầu 1 process với -it và thêm -rm vào để kill và remove nó ngay sau khi exits



