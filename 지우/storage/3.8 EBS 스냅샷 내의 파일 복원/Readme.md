# 3.8 EBS 스냅샷 내의 파일 복원

> _EBS 스냅샷에서 볼륨을 생성하고 EC2 인스턴스에 볼륨을 탑재한 뒤, 파일을 볼륨에 복사하여 복원_

<br>

① EC2 인스턴스에 연결된 EBS 볼륨 검색

```bash
EBS_ID=$(aws ec2 describe-volumes \
	--filters Name=attachment.instance-id,Values=$INSTANCE_ID \
	--output text \
	--query Volumes[0].Attachments[0].VolumeId)
```

② EBS 볼륨의 스냅샷 생성

```bash
SNAPSHOT_ID=$(aws ec2 create-snapshot \
	--volume-id $EBS_ID \
	--output text --query SnapshotId)
```

③ 스냅샷에서 볼륨 생성하고 `VOLUME_ID` 환경변수로 저장

```bash
SNAP_ID=$(aws ec2 create-volume \
	--snapshot-id $SNAPSHOT_ID \
	--size 8 \
	--availability-zone ap-northeast-2c \
	--output text --query VolumeId)
```

---

🥕 **유효성 검사** : 볼륨을 인스턴스의 `/dev/sdf`로 연결하고 `lsblk` 명령으로 볼륨 확인

```bash
aws ec2 attach-volume --volume-id $SANP_ID \
	--instance-id $INSTANCE_ID --device /dev/sdf

# volume status → 'attached'
aws ec2 describe-volumes \
	--volume-ids $SANP_ID

# ssm 연결
aws ssm start-session --target $INSTANCE_ID

# 볼륨 확인
lsblk

# 연결된 디스크 마운트할 폴더 생성 및 마운트
# 기본적으로 같은 파일 시스템 중복 마운트가 방지되어 있으므로 -o nouuid 매개 변수를 사용하여 동일 uuid로 볼륨 마운트
sudo mkdir /mnt/restore
sudo mount -t xfs -o nouuid /dev/nvme1n1p1 /mnt/restore

# 마운트된 볼륨에서 로컬 파일 시스템으로 파일 복사
sudo cp /mnt/restore/home/ec2-user/.bash_profile \
	/tmp/.bash_profile.restored

# 마운트 해제 및 로그아웃
sudo umount /dev/nvme1n1p1
exit
```
