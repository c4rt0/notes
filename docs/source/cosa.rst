CoreOS
===================================

I keep forgetting stuff, so here I will be placing the usefull CoreOS related commands.

alias
-----

adamsky@laptop Work/coreos-assembler (pr/testiscsi %) » podman run --rm -ti --security-opt=label=disable --privileged \
--uidmap=1000:0:1 --uidmap=0:1:1000 --uidmap=1001:1001:64536 \
-v=${PWD}:/srv/ --device=/dev/kvm --device=/dev/fuse \
--tmpfs=/tmp -v=/var/tmp:/var/tmp \
-v=/home/adamsky/Work/coreos-assembler-hacking/:/srv/fcos \
--name=cosa quay.io/coreos-assembler/coreos-assembler:latest shell

[coreos-assembler]$ cd fcos

[coreos-assembler]$ pwd
/srv/fcos

[coreos-assembler]$ ls
builds  cache  overrides  src  tmp

[coreos-assembler]$ ../mantle/build kola
Building kola

[coreos-assembler]$ ../bin/kola testiso -S iso-install-iscsi


===================================


cosa shell

./mantle/build kola


./bin/kola list | grep coreos.unique.boot.failure

./bin/kola run -b fcos --qemu-image fedora-coreos-38.20230918.dev.0-qemu.x86_64.qcow2 coreos.unique.boot.failure


[coreos-assembler]$ ./mantle/build kola
Building kola

[coreos-assembler]$ ./bin/kola run -b fcos --qemu-image fedora-coreos-38.20230918.dev.0-qemu.x86_64.qcow2 coreos.unique.boot.failure

podman run --rm -ti --security-opt=label=disable --privileged --uidmap=1000:0:1 --uidmap=0:1:1000 --uidmap=1001:1001:64536 -v=${PWD}:/srv/ --device=/dev/kvm --device=/dev/fuse --tmpfs=/tmp -v=/var/tmp:/var/tmp -v=/home/adamsky/Work/coreos-assembler-hacking/:/srv/fcos --name=cosa quay.io/coreos-assembler/coreos-assembler:latest shell

cosa kola 

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
		resultingError := inst.WaitAll(ctx)

		if resultingError == nil {
			resultingError = fmt.Errorf("Ignition unexpectedly succeeded")
		} else if resultingError == platform.ErrInitramfsEmergency {
			// Expectred initramfs failure, checking the console file to insure
			// that coreos.ignition.failure failed
			b, err := os.ReadFile(builder.ConsoleFile)
			if err != nil {
				resultingError = err
			}
			isExist, err := regexp.Match("/notwritable.txt", b)
			if err != nil {
				resultingError = err
			}
			if isExist {
				// The expected case
				resultingError = nil
			} else {
				resultingError = errors.Wrapf(err, "expected coreos.ignition.failure to fail")
			}
		} else {
			resultingError = errors.Wrapf(err, "expected initramfs emergency.target error")
		}
		errchan <- err
	}()

	select {
	case <-ctx.Done():
		if err := inst.Kill(); err != nil {
			return errors.Wrapf(err, "failed to kill the vm instance")
		}
		return errors.Wrapf(ctx.Err(), "timed out waiting for initramfs error")
	case err := <-errchan:
		if err != nil {
			return err
		}
		return nil
	}
}



//////////////////////////////////////////////////////

Co-authored-by: Adamn Piasecki <c4rt0gr4ph3r@redhat.com>
