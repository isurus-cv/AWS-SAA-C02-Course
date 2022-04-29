### Architecture
![image](https://user-images.githubusercontent.com/88237437/165974083-c8b1df7c-ad92-4974-b036-e5835671eba6.png)

### Demo
![image](https://user-images.githubusercontent.com/88237437/165974200-9772f767-7dbd-4cbb-a1f5-40f3c7d82b32.png)

Ping to the EC2 instances fails
![image](https://user-images.githubusercontent.com/88237437/165974287-fa05a24d-f699-4351-b085-9532fac62dbb.png)

#### Setup the CGW
![image](https://user-images.githubusercontent.com/88237437/165974434-abb85f39-328c-4568-8f22-7972d1c57040.png)

![image](https://user-images.githubusercontent.com/88237437/165974481-5aec14da-99fd-4d4c-ab32-72f9c2587bd4.png)

![image](https://user-images.githubusercontent.com/88237437/165974538-7bf6d797-2e1b-405e-9462-a1d4f85bb28b.png)

#### Setup the VGW
![image](https://user-images.githubusercontent.com/88237437/165974610-9eb3b201-dfb6-4700-9266-2046fecb3555.png)

![image](https://user-images.githubusercontent.com/88237437/165974642-9466e8e3-249e-4d8b-893b-089641907b28.png)

![image](https://user-images.githubusercontent.com/88237437/165974715-97d51cc2-e32e-4c02-8320-39e77f22fd94.png)

Attach the VPC
![image](https://user-images.githubusercontent.com/88237437/165974763-d72edd6f-c27b-46bc-b257-c46cc4848497.png)

![image](https://user-images.githubusercontent.com/88237437/165974794-9b4ed537-fce7-4e5b-bbd0-f1ff433b267f.png)

#### Setup the Site to Site VPN
![image](https://user-images.githubusercontent.com/88237437/165974885-b9a17b47-47b4-41ce-82de-63289a8b3e0c.png)

![image](https://user-images.githubusercontent.com/88237437/165974940-b853b5d7-7ec5-4f3f-aa6f-f32b45227114.png)
![image](https://user-images.githubusercontent.com/88237437/165974999-210f9937-325c-4bd6-b087-477a52b8aa81.png)
![image](https://user-images.githubusercontent.com/88237437/165975032-135b3aa0-b82b-4d5a-a7f8-dccb7534d152.png)

![image](https://user-images.githubusercontent.com/88237437/165975074-ce57b429-5e29-4d1a-91fb-5d2bd1c229e6.png)

Download the configuration to setup the network in on-premises side
![image](https://user-images.githubusercontent.com/88237437/165975187-525fa3d0-28ad-484c-9047-73cee161383c.png)

#### Setup the Network in the Client side
![image](https://user-images.githubusercontent.com/88237437/165975379-0c7d6d99-7312-4e54-a760-012f06fc68fd.png)

#### Add on-premises record to the route table of the VPC
![image](https://user-images.githubusercontent.com/88237437/165975541-674517ee-8f37-41d0-be67-a54501e79342.png)

![image](https://user-images.githubusercontent.com/88237437/165975593-f21b197b-f7e0-484d-8cbf-5bfd1ad2ab95.png)

#### Site to Site VPN is established
![image](https://user-images.githubusercontent.com/88237437/165975763-f9b963bf-b54a-4fa1-82ed-f7290776aefb.png)

#### Additional Steps
Enable route propagation to add routes to the Route table automatically
![image](https://user-images.githubusercontent.com/88237437/165975911-c5c429d8-2bbd-4a40-8ff9-6e689a628216.png)

![image](https://user-images.githubusercontent.com/88237437/165975971-74c5825f-e42e-4e85-8260-7ffd03ae27b7.png)

#### VPN Tunnels
![image](https://user-images.githubusercontent.com/88237437/165976111-7b92c58a-b104-45b3-b6fb-5d4bd5ee4551.png)


