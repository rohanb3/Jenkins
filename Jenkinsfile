node{

  /* Parameters as expected: */
  env.ARCHIVE_DIR    = params.ARCHIVE_DIR
  // A list of propertyfiles seperated by colon, each including the directory location:
  // No directory means the bin subdirectory of the Deployment Scripting Suite:
  env.ENVPROPFL      = params.ENVPROPFL
  env.PEGA_FIXDIR    = params.PEGA_FIXDIR
  env.REMOTE_WORKDIR = params.REMOTE_WORKDIR
  env.sshcon         = params.sshcon
  env.ssh_creds      = params.ssh_creds

  /********************************************
   *  SSH-AGENT scope - begin
   ********************************************/
  sshagent (credentials: [ssh_creds]) {

    try{

      // Distribute contents addressing the Weblogic:
      stage ('Distribute contents addressing the Weblogic'){
        sh script: '''\
${sshcon} "${REMOTE_WORKDIR}/bin/subdeploy.sh -e ${ENVPROPFL} eTop_wl.filemap"
'''
      }

      // Disable user access:
      stage ('Disable user access'){
        sh script: '''\
${sshcon} "${REMOTE_WORKDIR}/bin/db_set_user_access_level.sh -a disable -d ${ARCHIVE_DIR}/DB -e ${ENVPROPFL}"
'''
      }

      // Stop managed server via admin server:
      stage ('Stop Cluster'){
        sh script: '''\
${sshcon} "${REMOTE_WORKDIR}/bin/weblogic.sh -e ${ENVPROPFL} \
                                             cluster stop"
'''
      }

      // The database table update:
      stage ('Install database fix'){
        sh script: '''\
${sshcon} "${REMOTE_WORKDIR}/bin/install_DB.sh -d ${ARCHIVE_DIR}/DB -e ${ENVPROPFL} -s install_fix_99.sh"
'''
      }

      // PEGA_FIXDIR must be provided as parameter:
      stage ('Install PEGA fix jars'){
        sh script: '''\
${sshcon} "${REMOTE_WORKDIR}/bin/weblogic.sh -e ${ENVPROPFL} -p R4.5-F99-Fix-01 -1 \
                                             shell -C \'$${pega:prpc:import:workdir}\' \
                                                   -T \'$${pega:build:timeout}\' \
                                                   \'$${pega:prpc:import:call}\' pr \
                                                                     \'$${etop:appserver:subtree:rules}/$${PACKAGEDIR}\' \
                                                                                                               -x \'&&\' \
                                                   grep -g \'$${pega:confirm:build}\' \
                                          \'$${etop:appserver:subtree:rules}/$${PACKAGEDIR}/*import_$${PACKAGEDIR}.log\' \
                                                                                            -x \'2>\' /dev/null -x \'|\' \
                                                   tail -1"
'''
      }

      // Additional steps to adapt Weblogic:
      stage ('increase WTA timeout'){
        sh script: '''\
${sshcon} "${REMOTE_WORKDIR}/bin/weblogic.sh -e ${ENVPROPFL} \
                                             \'$${etop:appserver:subtree:weblogic}/update/adaptWtaTimeout.py\'"
'''
      }

      // Clear PRPC cache files:
      stage ('Clear PRPC cache') {
        sh script: '''\
${sshcon} "${REMOTE_WORKDIR}/bin/weblogic.sh -e ${ENVPROPFL} \
                                             shell -C \'$${etop:appserver:subtree:pegatmp}\' \
                                                   rm -rf PegaRULES_Extract_Marker.txt PRGen* StaticContent LLC"
'''
      }

	  // Restart managed server via admin server and check the success:
      stage ('Start Cluster with Pega log check'){      
        sh script: '''\
${sshcon} "${REMOTE_WORKDIR}/bin/weblogic.sh -e ${ENVPROPFL} \
                                             cluster start \
                                             shell -C \'$${etop:appserver:subtree:pegalogs}\' \
                                                   -T \'$${pega:start:timeout}\' \
                                                   grep -g \'$${pega:confirm:start}\' \
                                                        PegaRULES_\'$${env:ident}\'etop_MS1.log"
'''
      }

      // Check installed versions:
      stage ('Check installed versions') {
        sh script: '''\
${sshcon} "${REMOTE_WORKDIR}/bin/check_installed_versions.sh -d ${ARCHIVE_DIR}/DB -e ${ENVPROPFL}"
'''
      }

      // Enable user access:
      stage ('Enable user access'){      
        sh script: '''\
${sshcon} "${REMOTE_WORKDIR}/bin/db_set_user_access_level.sh -a enable -d ${ARCHIVE_DIR}/DB -e ${ENVPROPFL}"
'''
      }    

    } catch (ex) {
      ex.detailMessage();
      ex.printStackTrace();
    }
  }
}