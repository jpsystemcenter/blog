---
title: SCOM 2019 �ȍ~�� Windows ���o�E�B�U�[�h�Ō��o���s
date: 2022-08-03 15:00:00
tags:
  - SCOM
  - Troubleshoot
---

<!-- more -->
�F�l����ɂ��́ASystem Center �T�|�[�g�`�[���� ���� �ł��B

�{���� System Center Operations Manager�i�Ȍ� SCOM�j  2019 �ŕύX�̂������d�l�� 1 �����Љ�����܂��B
2022 �N 8 �����݂̏󋵂Ƃ��āA�i[SCOM 2012 R2�̉����T�|�[�g�̃t�F�[�Y��2022�N7���ɏI��](https://docs.microsoft.com/ja-jp/lifecycle/products/microsoft-system-center-2012-r2-operations-manager)�j�������Ƃɔ����ASCOM 2019 �� 2022 �Ȃǌ�p�o�[�W�����̊���V�K�\�z���Ĉڍs��i�߂Ă���A�������͂��v�悳��Ă��邨�q�l�����邩�Ƒ����܂��B

SCOM 2019 �̐V�K�����\�z��� Windows �R���s���[�^�[�����o����ۂ� SCOM �C���X�g�[�� ���[�U�ȊO�Ō��o�����݂�Ɖ��}�̃G���[�i�ΏۃR���s���[�^�[�����Ȃ��j�Əo�͂����ꍇ���������܂��B


���̎��ۂ͂���܂ł̃o�[�W�����ƈقȂ茟�o������s�A�J�E���g�ɑ΂��ĂăT�[�r�X ���O�I���A�N�Z�X����L���ɂ��Ȃ��ꍇ�ɔ������Ă���\�����������܂��B
�ݒ������Ă��Ȃ���΁A���L�菇�ɂĈ�x�ݒ�����������A���ۂ����P���邩���m�F����������΂Ƒ����܂��B

SCOM �Ǘ��T�[�o�[�ɂĈȉ��ݒ�����鎖�ŁA�T�[�r�X ���O�I���A�N�Z�X����L���ɂł��܂��B
1. SCOM �Ǘ��T�[�o�[�ɊǗ��ғ����ŃT�C���C�����܂��B
2. [�Ǘ��c�[��] �Ɉړ����A [���[�J�� �Z�L�����e�B �|���V�[] ���N���b�N���܂��B
�@�Ⴆ�΁AWindows Server 2016 �ȍ~�ł́A�X�^�[�g���j���[���J���A[Windows �Ǘ��c�[��] -> [���[�J�� �Z�L�����e�B �|���V�[] ���N���b�N���܂��B
3. [���[�J�� �|���V�[] ��W�J���A [���[�U�[�����̊��蓖��] ���N���b�N���܂��B
4. �E���̃E�B���h�E�ŁA [�T�[�r�X�Ƃ��ă��O�I��] ���E�N���b�N���A [�v���p�e�B] ��I�����܂��B
5. [���[�U�[�܂��̓O���[�v�̒ǉ�] �I�v�V�������N���b�N���A�ȉ��ǂ��炩��ǉ����܂��B
6. �T�[�o�[���ċN�����܂��B

�Q�l�T�C�g�Fhttps://docs.microsoft.com/ja-jp/system-center/scom/enable-service-logon?view=sc-om-2019#enable-service-log-on-permission-for-run-as-accounts



> �Ώۃo�[�W�����F
>�@System Center Operations Manager 2019
>�@System Center Operations Manager 2022